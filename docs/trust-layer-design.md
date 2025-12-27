# Trust Layer Design - Sentinels & Web-of-Trust

**Status:** Design Phase  
**Last Updated:** December 2025

---

## Overview

The PPA trust layer combines **expert human verification (Sentinels)** with **algorithmic web-of-trust scoring** to create a multi-tiered safety system that scales while maintaining accountability.

---

## The Trust Ladder (4 Levels)

### Level 0: New User (Unverified)

**Capabilities:**

- ✅ Browse limited feed (verified users only)
- ✅ Post low-risk service requests (public meeting locations)
- ❌ Cannot offer any services
- ❌ Cannot access exact addresses in listings

**Requirements to Advance:**

- Option A: Build social graph (10+ connections with L1+ users)
- Option B: Schedule hub verification appointment

**Rationale:** Prevents throwaway accounts from accessing vulnerable populations.

---

### Level 1: Community Connected

**How to Achieve:**

- Have 10+ mutual connections with L1+ users, OR
- Have trust score > 0.3 via Grapevine algorithm

**Capabilities:**

- ✅ Browse full feed (L1+ users)
- ✅ Post broader service requests
- ✅ Offer services at public locations only (libraries, community centers)
- ✅ Receive encrypted DMs from L2+ users
- ❌ Cannot offer in-home services

**Use Cases:**

- Tech help at library
- Tutoring at coffee shop
- Community garden volunteering
- Public event coordination

**Rationale:** Social graph provides basic accountability without requiring ID verification.

---

### Level 2: Hub Verified

**How to Achieve:**

- Attend in-person verification appointment at local PPA hub
- Sentinel reviews government-issued ID (not stored, just verified)
- Photo taken for badge (optional, user consent)

**Verification Process:**

1. User schedules appointment via app
2. Arrives at hub (senior center, library, church partner location)
3. Sentinel checks ID matches profile name
4. Optional: Basic background questions ("Why interested in PPA?")
5. Sentinel publishes NIP-58 verification badge to Nostr
6. User's trust level updates immediately across all Nostr apps

**Capabilities:**

- ✅ All L1 capabilities, plus:
- ✅ Offer in-home services (elder check-ins, light housework)
- ✅ Request in-home assistance
- ✅ Higher visibility in feeds (verification badge shown)
- ✅ Access to detailed service history of others

**Rationale:** In-person verification drastically reduces bad actors while remaining accessible (no fees, no credit check, no invasive background check).

---

### Level 3: Sentinel Verified (Enhanced)

**How to Achieve:**

- L2 status + Enhanced background check for high-risk services
- Categories requiring L3:
  - In-home care for elderly/disabled
  - Childcare/tutoring (in-home)
  - Financial assistance navigation
  - Medical transport

**Enhanced Verification Includes:**

- Criminal background check (coordinated by Sentinel, user pays ~$30)
- Reference checks (2-3 professional or character references)
- Category-specific certifications (CPR, First Aid, teaching license, etc.)
- Interview with Sentinel team

**Capabilities:**

- ✅ All L2 capabilities, plus:
- ✅ Offer highest-trust services
- ✅ Priority matching in feeds
- ✅ Gold shield verification badge (visible across Nostr)
- ✅ Can serve as hub representative (if approved)

**Rationale:** For vulnerable populations (elderly, disabled, children), additional vetting is non-negotiable.

---

## Sentinel Roles & Responsibilities

### Who Are Sentinels?

**Ideal Backgrounds:**

- Retired law enforcement, military (especially MPs, intelligence)
- Former EMTs, firefighters, nurses
- Software security professionals
- Private investigators
- HR professionals with screening experience
- Social workers with safeguarding training

**NOT Required:**

- Active law enforcement (prefer retired to avoid conflicts)
- Legal credentials (not lawyers, not judges)
- Technical expertise (we provide tools)

**Key Traits:**

- Deep understanding of risk assessment
- Experience with identity verification
- Strong ethical compass
- Community trust and respect
- Available for weekly verification clinics

---

### Sentinel Team Structure (Per Geographic Area)

**Lead Sentinel (1 per area):**

- Coordinates verification schedule
- Makes final calls on disputed cases
- Trains new Sentinels
- Reports to PPA core team
- Time commitment: 5-10 hours/week

**Verification Sentinels (2-4 per area):**

- Run weekly verification clinics
- Review IDs, conduct interviews
- Publish verification badges
- Time commitment: 2-4 hours/week

**Escalation Sentinel (1 per area):**

- Handles disputes and safety concerns
- Investigates reported incidents
- Coordinates with local authorities if needed
- Time commitment: On-call, ~2 hours/week average

---

### Sentinel Workflows

#### 1. Hub Verification (L2 → L3)

**Tools Provided:**

- Sentinel Dashboard (web app)
- ID scanning checklist (laminated card)
- NIP-58 badge signing tool
- Incident reporting form

**Process:**

```
1. User arrives at scheduled time
2. Sentinel logs into dashboard, pulls up appointment
3. Reviews ID:
   - Name matches profile
   - Photo matches person
   - ID not expired
   - No obvious forgery signs
4. Brief interview:
   - "Why interested in PPA?"
   - "What services do you want to offer/request?"
   - "Have you used similar platforms before?"
5. Sentinel makes decision:
   - APPROVE → Publish L2 badge immediately
   - DEFER → Request additional info, reschedule
   - DENY → Document reason, notify user (rare)
6. User sees badge update in app within seconds
```

**Decision Criteria:**

- ID verification: Pass/Fail (objective)
- Interview assessment: Sentinel discretion (subjective)
- Red flags trigger DEFER or DENY:
  - Evasive answers
  - Inconsistencies in story
  - Hostile/aggressive behavior
  - Known criminal history for violent crimes (flagged by background check if done)

#### 2. Enhanced Verification (L2 → L3)

**Additional Steps:**

```
1. User initiates L3 upgrade in app
2. Pays background check fee (~$30, covers cost only)
3. Sentinel requests references via app
4. Reference checks conducted (phone/email)
5. Background check results reviewed
6. If approved: Sentinel publishes L3 badge
7. If denied: User notified with explanation, can appeal
```

**Background Check Criteria:**

- Violent crimes: Automatic disqualification
- Property crimes: Case-by-case (how recent, severity)
- Drug offenses: Case-by-case (rehabilitation evidence)
- No criminal record: Automatic pass

#### 3. Dispute Resolution

**Trigger:** User reports concern about another user

**Process:**

```
1. Report submitted via app (who, what, when, evidence)
2. Escalation Sentinel notified immediately
3. Sentinel reviews:
   - Completion history of both parties
   - Previous reports (if any)
   - Encrypted DM thread (if consent given)
4. Sentinel makes decision:
   - NO ACTION: Misunderstanding, no pattern
   - WARNING: First-time minor issue, logged
   - SUSPENSION: Temporary (7-30 days), badge removed
   - BAN: Permanent, pubkey blacklisted
5. Both parties notified of decision
6. Appeals process available (Lead Sentinel reviews)
```

**Escalation Triggers:**

- Safety concern (immediate Sentinel review)
- Repeated no-shows (3+ without cancellation)
- Aggressive/threatening behavior
- Scam attempt
- Violation of terms (spam, harassment)

---

## Web-of-Trust (Algorithmic Layer)

### Grapevine Integration (Planned Q2 2025)

**What is Grapevine?**
A trust inference algorithm that calculates how much you should trust someone based on your social graph.

**How It Works:**

1. You explicitly trust certain people (direct follows on Nostr)
2. Those people trust others
3. Grapevine infers trust scores for people 2-3 degrees away
4. Scores decay with distance and time

**Example:**

```
You → Alice (trust: 1.0)
Alice → Bob (trust: 0.8)
Bob → Charlie (trust: 0.7)

Your trust score for Charlie = 1.0 * 0.8 * 0.7 = 0.56
```

**PPA Implementation:**

- Users can set minimum trust threshold (default: 0.3)
- Feed shows only users above threshold
- Can override for specific users ("always show" or "never show")

**Benefits:**

- Personalized safety (your network, your rules)
- Resistant to Sybil attacks (fake accounts)
- No central authority deciding who's trustworthy

---

### Simple Trust Score (MVP - Before Grapevine)

**Calculation:**

```javascript
function calculateTrustScore(userPubkey, viewerPubkey) {
  let score = 0.0;

  // Verification badges
  if (hasL3Badge(userPubkey)) score += 0.5;
  else if (hasL2Badge(userPubkey)) score += 0.3;

  // Social graph
  if (directFollow(viewerPubkey, userPubkey)) score += 0.3;
  else if (twoDegreesAway(viewerPubkey, userPubkey)) score += 0.1;

  // Completion history
  const completions = getCompletedServices(userPubkey);
  score += Math.min(completions.length * 0.02, 0.2); // Max +0.2

  return Math.min(score, 1.0); // Cap at 1.0
}
```

**Feed Filtering:**

- Show users with score ≥ 0.3 (configurable)
- Sort by score descending
- Highlight L3 users prominently

---

## Safety Features

### 1. Progressive Disclosure

- Public listings show only: geohash, task summary, timeframe
- Exact address shared ONLY in encrypted DM after match
- Phone numbers optional, shared after mutual confirmation

### 2. Completion Verification

Both parties must confirm service completion:

```
Helper: "I completed this task"
Requester: "I confirm task completed"
→ Reputation points awarded, visible to all
```

Mismatched confirmations trigger Sentinel review.

### 3. Rating System (Future)

- 5-star scale (like Uber)
- Tied to specific completed services (can't game it)
- Visible to L2+ users only
- Comment required for <3 stars (prevents drive-by negativity)

### 4. Emergency Contact

- Users can add emergency contact (encrypted in DM metadata)
- "Check-in" feature: Notify contact when meeting someone new
- Panic button (future): Immediately notify Sentinel + emergency contact

---

## Privacy Protections

### Data Minimization

**What Sentinels See:**

- Name and photo on ID (during verification only, not stored)
- Nostr pubkey
- Service request history (if user consents)

**What Sentinels DON'T See:**

- Exact home addresses
- Phone numbers
- Financial info
- Medical history
- Unrelated private messages

### Decentralized Storage

- Verification badges published to Nostr (public, permanent)
- Personal data stored encrypted in DynamoDB (access controlled)
- Encrypted DMs readable only by sender/receiver
- Background check results stored locally by Sentinel (not uploaded)

### Right to Deletion

- User can request account deletion
- Verification badges remain on Nostr (signed events can't be "deleted" from relays)
- But: User stops publishing, so new apps won't see old badges
- Encrypted data in DynamoDB permanently deleted

---

## Geographic Distribution

### Hub Locations (Pilot Phase)

**Criteria for Hub Partners:**

- Existing community trust (senior centers, libraries, churches)
- Accessible location (public transit, parking, ADA compliant)
- Weekly availability (2-4 hour verification clinics)
- Space for private interviews (not completely public)

**Pilot Cities (Example):**

- Lawrence, MA (ZIP: 01841, 01843)
- Partner: Senior center on Broadway
- Sentinels: 1 Lead, 2 Verifiers, 1 Escalation

**Expansion Model:**

- Prove concept in 1-2 cities (6 months)
- Document playbook (Sentinel recruitment, training, workflows)
- Open applications for new hubs (via GiveSendGo, community partnerships)
- Target: 10-20 cities by end of Year 1

---

## Sentinel Compensation (Future)

**Current:** Volunteer-based (pilot phase)

**Planned (Post-Funding):**

- Lead Sentinel: $500/month stipend
- Verification Sentinels: $50 per clinic (2-4 hours)
- Escalation Sentinel: $200/month + per-incident fees
- Token rewards: PPC allocation for early Sentinels (retroactive)

**Why Volunteer First:**

- Attracts mission-driven individuals
- Proves viability before financial commitment
- Builds culture of service (not just gig work)

---

## Open Questions to Resolve

1. **Background Check Provider:** Which API? (Checkr, Certn, GoodHire?)
2. **Sentinel Training:** In-person workshop or video modules?
3. **Liability Insurance:** Do Sentinels need coverage? Does PPA?
4. **Appeals Process:** How many levels? Who's the final authority?
5. **Cross-Hub Recognition:** If verified in Lawrence, trusted in Boston?

---

## Success Metrics (60-Day Pilot)

**Trust Layer:**

- 100+ L2 verifications completed
- 10+ L3 enhanced verifications
- <5% dispute rate
- 0 major safety incidents

**Sentinel Performance:**

- Average verification time: <15 minutes
- Appointment availability: <7 days wait
- User satisfaction: >80% "felt safe and respected"

**Social Graph:**

- 50+ users with trust score >0.5
- Average 15 connections per user
- 80% of matches within 2 degrees of separation

---

## Resources

- **Grapevine Protocol:** [github.com/mplorentz/grapevine](https://github.com/mplorentz/grapevine)
- **NIP-58 (Badges):** [github.com/nostr-protocol/nips/blob/master/58.md](https://github.com/nostr-protocol/nips/blob/master/58.md)
- **Web of Trust Research:** [freehaven.net/anonbib/cache/wot.pdf](http://freehaven.net/anonbib/cache/wot.pdf)
