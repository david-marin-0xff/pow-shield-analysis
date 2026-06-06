# PoW-Shield Analysis

> This repository is my technical analysis of the original PoW-Shield project.

## Original Project

Original repository:

https://github.com/RuiSiang/PoW-Shield

Project description:

"Project dedicated to fight Layer 7 DDoS with proof of work, with an additional WAF and controller. Completed with full set of features and containerized for rapid and lightweight deployment."

---

# Purpose of This Repository

This repository serves as a learning journal and technical analysis of PoW-Shield.

Goals:

- Understand the architecture of the project.
- Learn how proof-of-work is used for request validation.
- Study how a reverse proxy can protect backend applications.
- Analyze rate limiting, WAF, and security features.
- Document findings and open questions.
- Explore ideas for future projects inspired by this design.

This repository is not intended to replace or compete with the original project.

---

# High-Level Understanding

Based on the project documentation, PoW-Shield appears to function as a reverse proxy placed in front of a web application.

Conceptually:

```text
Client
   │
   ▼
PoW-Shield
   │
   ▼
Backend Application
```

All incoming traffic reaches PoW-Shield first.

PoW-Shield then determines whether:

- The request should be allowed.
- A proof-of-work challenge should be issued.
- The request should be rate-limited.
- The request should be blocked.
- Additional WAF rules should be applied.

---

# What Is Proof-of-Work?

Proof-of-work (PoW) is a mechanism that requires a client to perform a computational task before gaining access.

General concept:

1. Server generates a challenge.
2. Client computes a valid solution.
3. Client submits the solution.
4. Server verifies the solution.
5. Access is granted.

The goal is to make large-scale automated abuse more expensive.

---

# Expected Request Flow

Current understanding:

```text
User visits protected site
            │
            ▼
      PoW-Shield
            │
            ▼
Challenge required?
      │           │
     Yes          No
      │           │
      ▼           ▼
 Serve Challenge  Forward Request
      │
      ▼
 Browser Solves Challenge
      │
      ▼
 Submit Solution
      │
      ▼
 Verification
      │
      ▼
 Access Granted
```

---

# Main Components Identified

## Reverse Proxy

Acts as an intermediary between clients and the backend application.

Potential responsibilities:

- Traffic inspection
- Request forwarding
- Authentication checks
- Security controls

---

## Proof-of-Work Engine

Responsible for:

- Challenge generation
- Challenge verification
- Difficulty enforcement

Questions:

- How are challenges generated?
- How is difficulty determined?
- Are challenges time-limited?

---

## WAF (Web Application Firewall)

Appears to provide additional protection.

Areas to investigate:

- Rule engine
- Signature detection
- Custom rules
- Attack pattern matching

---

## Rate Limiting

Likely responsible for:

- Request counting
- Burst detection
- Abuse prevention

Areas to investigate:

- Per-IP limits
- Sliding window algorithms
- Temporary bans

---

## Controller / Dashboard

Appears to provide management functionality.

Areas to investigate:

- Metrics
- Monitoring
- Configuration management
- Logging

---

## Redis Support

Documentation suggests support for Redis.

Possible uses:

- Session storage
- Distributed synchronization
- Shared state between instances

Areas to investigate:

- Which data is stored?
- How synchronization works?
- Failover behavior

---

# Deployment Model

Expected deployment:

```text
Internet
    │
    ▼
PoW-Shield
    │
    ▼
Nginx / Web Application
    │
    ▼
Backend Services
```

Need to verify actual architecture from source code.

---

# Security Concepts To Research

- Reverse proxies
- Layer 7 attacks
- Proof-of-work systems
- Hashcash
- Rate limiting algorithms
- Web application firewalls
- Session validation
- Challenge-response systems

---

# Open Questions

Questions to answer while studying the code:

- How is proof-of-work implemented?
- Which hashing algorithm is used?
- How are clients tracked?
- How are sessions managed?
- How does Redis integrate?
- How does the WAF operate?
- How are rules configured?
- What metrics are collected?
- How does scaling work?

---

# Ideas Inspired by This Project

Potential future project:

Adaptive Scraping Gateway

Concept:

Protect selected endpoints rather than an entire website.

Example:

```text
/api/*
/downloads/*
/search/*
```

Potential features:

- Adaptive proof-of-work
- Route-based protection
- Rate limiting
- Browser fingerprinting
- Analytics dashboard

This section will evolve as understanding of PoW-Shield improves.

---

# Study Log

## Session 1

Date:

Notes:

- Created analysis repository.
- Reviewed project description.
- Established initial understanding of architecture.
- Created list of research questions.

Next Steps:

- Examine repository structure.
- Identify entry point.
- Trace request flow.
- Locate proof-of-work implementation.
- 
