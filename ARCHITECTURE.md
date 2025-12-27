# PPA Prototype Build Specification

**For:** Development Team  
**Purpose:** Align on what we're actually building

---

## What We're Building

Community coordination infrastructure that makes mutual aid **predictable and trustworthy** - like how dating apps make meeting strangers safe through profiles/verification, or how Uber makes getting in a stranger's car normal through ratings/tracking.

**Core pattern:**

- Someone posts a **bounded request** (specific task, timeframe, outcome)
- Verified helpers **commit with clarity** (what they'll do, when, how long)
- **Trust signals** make both sides comfortable (verification badges, social graph, completion history)
- **Private coordination** happens after public matching

**Why this works:**

- Replaces vague volunteering ("help sometime?") with structured service packages
- Trust layer (Sentinels + web-of-trust) makes vulnerable populations feel safe
- Decentralized foundation (Nostr) means the network outlives any single organization
- Reusable for ANY coordination scenario (elder care, homeless services, civic engagement, skill exchanges)

---

## Technical Stack

### Frontend: Next.js + NDK

- ✅ `@nostr-dev-kit/ndk` v2.14.33
- ✅ `@nostr-dev-kit/ndk-react` for React integration
- ✅ Works with Next.js 15 App Router

### Backend: Hybrid

- ✅ **strfry relay** (self-hosted on EC2)
- ✅ **AWS Lambda** (existing infrastructure)
- ✅ **DynamoDB** (caching layer)
- ✅ **S3** (backups)

### Nostr Standards:

- ✅ NIP-07 (Browser extension login)
- ✅ NIP-99 (Classified listings - kind 30402)
- ✅ NIP-17 (Encrypted DMs)
- ✅ NIP-58 (Badges for Sentinel verification)
- ✅ NIP-13 (Proof-of-work spam prevention)

### Trust Layer:

- ⚠️ **Custom implementation** using NDK's follow graph queries
- Start simple: 2-degree separation trust scoring
- Can enhance later with PageRank-style algorithms

---

## Core User Flows

### 1. Sign Up & Verify

- User creates account (email/password custodial OR browser extension self-custody)
- Nostr keypair generated (stored custodial or user-managed)
- Optional: Schedule verification appointment at local hub
- Sentinel reviews ID → publishes verification event to Nostr
- User gets trust badge visible across all Nostr apps

### 2. Post Request

- Fill form: task type, location (geohash, not exact address), timeframe, outcome definition
- Published as NIP-99 event with tags: `#ppa`, `#task-type`, `#geohash`
- Broadcast to public relays + PPA private relay
- Cached in DynamoDB for fast filtering

### 3. Browse & Match

- Feed shows only: PPA-tagged posts, in user's geo-radius, from verified users (configurable trust threshold)
- Click "I can help" → initiates NIP-17 encrypted DM thread
- Private details (exact address, phone, payment) shared only in DM
- Both parties confirm commitment

### 4. Complete & Rate

- Helper marks task complete → creates completion event
- Requester confirms → creates confirmation event
- Both events contribute to reputation scores (Grapevine input)
- Optional: Sentinel reviews outcome for quality/safety

---

## Trust Ladder (Enforced by App Logic)

**L0 - New User**

- Can browse limited feed
- Can post low-risk requests only
- Cannot offer services requiring home access

**L1 - Community Connected**

- Has web-of-trust connections (Grapevine score > threshold)
- Can offer public-meeting services (tech help at library, etc.)

**L2 - Hub Verified**

- In-person ID check at local hub (no ID stored, just verification event)
- Can offer broader service categories

**L3 - Sentinel Verified**

- Enhanced background check for high-risk categories (in-home care, vulnerable populations)
- Gets priority visibility in feeds
- Trust badge portable across Nostr ecosystem

---

## Pilot Configuration Template

**Geographic Scope:** [2-3 ZIP codes]  
**Hub Partner:** [Senior center / Library / Church / Community org]  
**Sentinels:** [2-5 named roles: Lead, Ops, Training]  
**Service Package:** [Bounded task types with timebox + outcome definition]  
**Verification Schedule:** [Weekly clinic hours at hub]

**Success Metrics (60 days):**

- X verified users
- Y completed matches
- 0 safety incidents
- <48hr median match time
- 80%+ "clear expectations" score

---

## What We're NOT Building (Yet)

- Payment processing (comes after trust/matching proven)
- PPC token distribution (future layer)
- Advanced background check API integration (manual Sentinel process first)
- Mobile apps (web-first MVP)
- Complex rating algorithms (simple completion verification first)

---

## Why This Architecture Works

**Decentralized where it matters:**

- User identity (Nostr keys)
- Request/offer data (relay network)
- Trust signals (verification events)
- → Can't be deplatformed, data is portable

**Centralized where it helps:**

- Fast search/filtering (DynamoDB cache)
- Sentinel workflows (Lambda + admin dashboard)
- Spam prevention (curated relay + app-level filtering)
- → Good UX, controlled quality

**Result:** Feels like a normal app (familiar UX) but runs on censorship-resistant infrastructure (Nostr foundation).

---

## Immediate Technical Questions to Resolve

1. **NDK version** - Current stable is 2.x, confirm compatibility with Next.js 15
2. **Relay setup** - Use existing public relays only, or spin up `strfry` on EC2?
3. **Grapevine integration** - Library exists, need API/implementation details
4. **NIP-99 adoption** - Verify current relay support for classified listing events
5. **Custodial key management** - Encryption strategy for user keys in DynamoDB?
