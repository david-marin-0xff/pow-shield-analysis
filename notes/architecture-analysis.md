# PoW-Shield Architecture Analysis

> Personal analysis and reverse-engineering notes for the PoW-Shield project.
>
> Original Project:
> https://github.com/RuiSiang/PoW-Shield

---

# Analysis Status

| Component | Status |
|------------|---------|
| High-Level Architecture | 🟢 Understood |
| Reverse Proxy Flow | 🟢 Partially Understood |
| Proof-of-Work System | 🟢 Partially Understood |
| Rate Limiting | 🟢 Partially Understood |
| Redis Storage | 🟢 Partially Understood |
| WAF | 🟡 Needs Investigation |
| PoW Phalanx Controller | 🟢 Initial Understanding |
| Dashboard | 🔴 Not Investigated |
| Multi-Instance Sync | 🟡 Needs Investigation |

---

# Project Purpose

PoW-Shield is designed to protect web applications from Layer 7 attacks by requiring clients to solve a proof-of-work challenge before accessing protected resources.

The project also includes:

- Web Application Firewall (WAF)
- Rate Limiting
- IP Blacklisting
- Redis-backed state storage
- SSL Support
- Multi-instance synchronization
- Remote management via PoW Phalanx

---

# High-Level Architecture

Current understanding:

```text
                    ┌───────────────┐
                    │ PoW Phalanx   │
                    │ Controller    │
                    └───────┬───────┘
                            │
                       Socket.IO
                            │
                            ▼

┌─────────┐      ┌─────────────────┐
│ Client  │─────▶│   PoW Shield    │
└─────────┘      └───────┬─────────┘
                         │
            ┌────────────┼────────────┐
            │            │            │
            ▼            ▼            ▼

       Proof-of-Work    WAF      Rate Limiter
            │            │            │
            └────────────┼────────────┘
                         │
                         ▼

                      Redis
                         │
                         ▼

                  Backend Service
```

---

# Request Flow

Expected flow:

```text
Client Request
       │
       ▼
 Controller Middleware
       │
       ▼
 Blacklist Check
       │
       ▼
 WAF Scan
       │
       ▼
 Session Authorized?
       │
 ┌─────┴─────┐
 │           │
 No         Yes
 │           │
 ▼           ▼
 PoW       Rate Limit
 Page         │
 │            ▼
 ▼       Backend Proxy
 Solve
 Challenge
 │
 ▼
 Verification
 │
 ▼
 Authorized Session
```

---

# Proof-of-Work System

## Current Findings

The proof-of-work challenge appears to consist of:

- Prefix
- Difficulty

The client must compute a valid nonce.

The nonce is verified by the server.

---

## Difficulty

Configuration values:

```env
INITIAL_DIFFICULTY=13
DIFFICULTY=13
NONCE_VALIDITY=60000
```

README notes:

> Difficulty represents the number of leading zero bits required in the resulting hash.

Examples:

```text
0    = trivial
13   = default
256  = practically impossible
```

According to the documentation:

```text
Difficulty 13 ≈ 5 seconds of browser computation
```

---

## Important Observation

Earlier versions appear to initialize:

```ts
new Pow(config.initial_difficulty)
```

Newer code initializes:

```ts
new Pow()
```

and retrieves difficulty from configuration.

This suggests difficulty is now centrally controlled rather than stored within the PoW object.

---

# PoW Verification Flow

Current understanding:

```text
Server Generates Prefix
        │
        ▼
Client Receives Challenge
        │
        ▼
Client Searches For Nonce
        │
        ▼
Hash(prefix + nonce)
        │
        ▼
Leading Zero Bits >= Difficulty ?
        │
 ┌──────┴──────┐
 │             │
 No           Yes
 │             │
 ▼             ▼
 Retry      Submit
                │
                ▼
           Verification
                │
                ▼
           Session Authorized
```

---

# Redis Usage

Recent code changes indicate Redis is heavily used.

Observed key patterns:

## Bans

```text
ban:<ip>
```

Example:

```text
ban:192.168.1.10
```

---

## Whitelist Tokens

```text
wht:<token>
```

Example:

```text
wht:123e4567-e89b-12d3-a456-426614174000
```

---

## Request Statistics

```text
req:<ip>
```

Example:

```text
req:203.0.113.10
```

---

## Global Statistics

```text
stats:ttl_req
stats:legit_req
stats:bad_nonce
stats:ttl_waf
```

---

# WAF Findings

The WAF appears to inspect:

- URL
- Headers
- Body

The scan function can return:

```text
Rule ID
Rule Type
Rule Description
```

Statistics are collected for WAF detections.

Observed statistic:

```text
stats:ttl_waf
```

Questions:

- Where are WAF rules stored?
- Are rules configurable?
- Are custom rules supported?

---

# Rate Limiting Findings

Observed configuration:

```env
RATE_LIMIT=on
```

Observed logic:

- Request counters are maintained.
- Thresholds trigger bans.
- Bans are stored in Redis.
- Authorized sessions are tracked separately.

Need to investigate:

- Sliding window?
- Fixed window?
- Token bucket?

---

# PoW Phalanx Controller

Major discovery.

PoW Shield supports a controller called:

```text
PoW Phalanx
```

Connection method:

```text
Socket.IO
```

Observed commands:

```text
set_config
add_whitelist
remove_whitelist
ban
fetch_stats
update_model
```

This enables:

- Remote configuration
- Multi-instance coordination
- Dynamic difficulty adjustment
- Statistics collection

---

# Whitelist Tokens

Observed behavior:

Requests containing:

```http
PoW-Token: <uuid>
```

or

```text
?pow_token=<uuid>
```

can bypass proof-of-work if the token exists in Redis.

This may be intended for:

- Trusted clients
- Internal services
- Administrative access

---

# Open Questions

## Proof-of-Work

- Which hash function is used?
- How are prefixes generated?
- How is nonce validity enforced?

## Redis

- What is the expiration policy?
- How are counters reset?

## Controller

- How does PoW Phalanx authenticate?
- Can commands be replayed?

## WAF

- What rules are included by default?
- Can rules be added dynamically?

## Proxy Layer

- Which middleware performs backend forwarding?
- How are headers handled?

---

# Ideas Inspired By This Project

Potential future project:

## Adaptive Scraping Gateway

Differences from PoW-Shield:

### PoW-Shield

```text
Protect entire application
```

### Adaptive Scraping Gateway

```text
Protect selected endpoints only
```

Examples:

```text
/api/*
/downloads/*
/search/*
```

Potential Features:

- Route-specific protection
- Adaptive difficulty
- Browser fingerprinting
- Analytics dashboard
- Redis support
- C++ implementation

---

# Study Log

## Session 1

Date: 2026-06-06

Findings:

- PoW Shield acts as a reverse proxy.
- Redis is used as central state storage.
- Dynamic difficulty appears controller-driven.
- PoW Phalanx manages instances via Socket.IO.
- WAF statistics are collected.
- Whitelist tokens can bypass PoW.

Next Investigation:

1. Locate hash implementation.
2. Trace nonce verification path.
3. Identify proxy forwarding mechanism.
4. Investigate WAF rule storage.
5. Analyze session authorization lifecycle.
