# Poor People App - Technical Overview

**Community Coordination Infrastructure for Mutual Aid**

[![Status](https://img.shields.io/badge/status-prototype-yellow)](https://github.com/FotiosMpouris/poor-people-app-overview)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

---

## 🎯 What We're Building

Community coordination infrastructure that makes mutual aid **predictable and trustworthy** - combining the trust signals of dating apps with the clarity of Uber's service delivery.

**The Core Innovation:**

- **Dating Apps** taught us how to trust strangers through profiles, verification, and social graphs
- **Uber** taught us how to get clear expectations: what, when, how much, tracked progress
- **PPA** combines both: trust-based matching + bounded service commitments + decentralized infrastructure

### Why This Matters

Traditional volunteer coordination fails because it's vague ("can you help sometime?") and lacks accountability. PPA creates **structured service packages** where:

- Someone posts a **bounded request** (specific task, timeframe, outcome)
- Verified helpers **commit with clarity** (what they'll do, when, how long)
- **Trust signals** make both sides comfortable (verification badges, social graph, completion history)
- **Private coordination** happens after public matching

---

## 🏗️ Technical Architecture

📊 **[View System Architecture Diagram](diagrams/system-architecture.md)** | **[Trust Ladder](diagrams/trust-ladder.md)** | **[User Flow](diagrams/user-flow.md)**

### Stack

**Frontend:**

- Next.js 15 + React
- NDK (Nostr Development Kit) v2.14.33
- TypeScript + Tailwind CSS

**Backend (Hybrid):**

- Self-hosted Nostr relay (strfry on EC2)
- AWS Lambda (coordination logic)
- DynamoDB (caching layer)
- S3 (backups)

**Decentralization Layer:**

- Nostr protocol (censorship-resistant identity & data)
- NIP-07 (browser extension login)
- NIP-99 (classified listings - service requests)
- NIP-17 (encrypted DMs)
- NIP-58 (verification badges)
- NIP-13 (proof-of-work spam prevention)

### Why Hybrid Architecture?

**Decentralized where it matters:**

- User identity (Nostr keys - portable across apps)
- Request/offer data (relay network - can't be censored)
- Trust signals (verification events - permanent record)
