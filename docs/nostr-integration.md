# Nostr Integration - Technical Implementation Guide

**Status:** In Development  
**Last Updated:** December 2025

---

## Overview

PPA uses the Nostr protocol as its decentralized identity and data layer. This ensures user data portability, censorship resistance, and interoperability with the broader Nostr ecosystem.

---

## NIPs (Nostr Implementation Possibilities) We're Using

### ✅ NIP-01: Basic Protocol Flow

**Status:** Implemented  
**Purpose:** Core event structure and relay communication

**Implementation:**

- All PPA events follow standard Nostr event format (id, pubkey, created_at, kind, tags, content, sig)
- Events signed with user's private key (nsec)
- Published to multiple relays for redundancy

**Key Decisions:**

- Users can choose custodial (PPA manages keys) or self-custody (browser extension)
- Custodial keys encrypted in DynamoDB using AWS KMS

---

### ✅ NIP-07: Browser Extension Signer

**Status:** Planned Q1 2025  
**Purpose:** Allow users to sign events with extensions like Alby, nos2x

**Implementation:**

```javascript
// Check for NIP-07 support
if (window.nostr) {
  const pubkey = await window.nostr.getPublicKey();
  const signedEvent = await window.nostr.signEvent(event);
}
```

**Benefits:**

- Users never expose private keys to PPA
- Works with existing Nostr identity
- Portable across all Nostr apps

---

### ✅ NIP-99: Classified Listings (Service Requests)

**Status:** Core to MVP - In Development  
**Purpose:** Structured format for service requests/offers

**Event Structure (kind 30402):**

```json
{
  "kind": 30402,
  "tags": [
    ["d", "unique-listing-id"],
    ["title", "Need help with grocery shopping"],
    ["summary", "Weekly grocery run for elderly resident"],
    ["published_at", "1703001600"],
    ["location", "geohash-u4pruydqqvj"],
    ["price", "0", "USD"],
    ["t", "grocery"],
    ["t", "elder-care"],
    ["t", "ppa"]
  ],
  "content": "Detailed description of task, expectations, timeframe..."
}
```

**PPA-Specific Tags:**

- `#ppa` - All PPA listings tagged for filtering
- `#task-type` - Category (elder-care, tutoring, tech-help, etc.)
- `#geohash` - Privacy-preserving approximate location
- `#trust-level-required` - Minimum trust ladder level (L0-L3)

**Why NIP-99:**

- Standard format works across Nostr marketplace apps
- Others can build alternative UIs for same data
- Future: Lightning payments via `bolt11` tags

---

### ✅ NIP-17: Private Direct Messages

**Status:** Critical - In Development  
**Purpose:** Encrypted coordination after initial match

**Flow:**

1. User A posts service request (public NIP-99 event)
2. User B clicks "I can help" (triggers NIP-17 DM)
3. Private conversation shares:
   - Exact address (not in public listing)
   - Phone number
   - Specific scheduling details
   - Payment arrangements (future)

**Implementation:**

```javascript
// Using NDK
import { NDKPrivateKeySigner } from "@nostr-dev-kit/ndk";

const dm = new NDKEvent(ndk);
dm.kind = 14; // Encrypted DM
dm.content = await signer.encrypt(recipientPubkey, message);
dm.tags = [["p", recipientPubkey]];
await dm.publish();
```

**Security Considerations:**

- Messages encrypted with recipient's public key
- PPA relay stores encrypted messages only
- Users with self-custody keys can decrypt in any Nostr client

---

### ✅ NIP-58: Badges (Sentinel Verification)

**Status:** Planned Q1 2025  
**Purpose:** Portable trust badges issued by Sentinels

**Badge Types:**

- **Hub Verified (L2):** Green checkmark, issued by local hub Sentinel
- **Sentinel Verified (L3):** Gold shield, enhanced background check
- **Service Specialist:** Category-specific expertise (CPR certified, licensed tutor, etc.)

**Badge Event (kind 30009):**

```json
{
  "kind": 30009,
  "tags": [
    ["d", "ppa-hub-verified"],
    ["name", "PPA Hub Verified"],
    ["description", "Identity verified at local PPA hub"],
    ["image", "https://poorpeople.app/badges/hub-verified.png"],
    ["thumb", "https://poorpeople.app/badges/hub-verified-thumb.png"]
  ],
  "content": "Badge definition"
}
```

**Award Event (kind 8):**

```json
{
  "kind": 8,
  "tags": [
    ["a", "30009:sentinel-pubkey:ppa-hub-verified"],
    ["p", "user-pubkey"],
    ["verified-at", "Lawrence, MA Hub"],
    ["verified-date", "2025-01-15"]
  ],
  "content": "John Doe verified at Lawrence Hub on 2025-01-15"
}
```

**Why This Matters:**

- Badge follows user across ALL Nostr apps
- Sentinels build reputation as trusted verifiers
- Can't be faked (cryptographically signed by Sentinel)

---

### ✅ NIP-13: Proof of Work (Spam Prevention)

**Status:** Planned Q2 2025  
**Purpose:** Make spam economically expensive

**How It Works:**
Users must perform computational work to publish events. Difficulty increases for:

- New accounts (L0 users)
- High-volume posting
- Suspicious patterns

**Implementation:**

```javascript
// Require PoW difficulty of 20 for new users
const powTag = ["nonce", nonce, "20"];
event.tags.push(powTag);

// Mine the event
while (true) {
  event.id = getEventHash(event);
  if (event.id.startsWith("00000")) break; // 20 bits = 5 hex zeros
  nonce++;
}
```

**Adaptive Difficulty:**

- L0 users: Difficulty 20 (requires ~1 second on average CPU)
- L1-L2 users: Difficulty 10
- L3 users: No PoW required

---

## NDK (Nostr Development Kit) Integration

**Library:** `@nostr-dev-kit/ndk` v2.14.33  
**React Integration:** `@nostr-dev-kit/ndk-react`

### Why NDK?

- **Abstraction:** Handles relay management, subscription pooling, caching
- **React Hooks:** Easy integration with Next.js
- **Type Safety:** Full TypeScript support
- **Active Development:** Well-maintained by Nostr ecosystem

### Key NDK Features We're Using:

**1. Relay Pool Management:**

```javascript
import NDK from "@nostr-dev-kit/ndk";

const ndk = new NDK({
  explicitRelayUrls: [
    "wss://relay.poorpeople.app", // Our strfry relay
    "wss://relay.damus.io",
    "wss://nos.lol",
    "wss://relay.nostr.band",
  ],
});

await ndk.connect();
```

**2. Event Subscriptions:**

```javascript
// Subscribe to PPA service requests
const filter = {
  kinds: [30402],
  "#t": ["ppa"],
  "#geohash": ["u4pruyd"], // Lawrence, MA area
};

const sub = ndk.subscribe(filter);
sub.on("event", (event) => {
  // Handle new service request
});
```

**3. User Profiles (NIP-05):**

```javascript
const user = ndk.getUser({ pubkey });
await user.fetchProfile();
console.log(user.profile.name, user.profile.picture);
```

---

## Relay Strategy

### Primary Relay: Self-Hosted strfry

**URL:** `wss://relay.poorpeople.app`  
**Purpose:** Guaranteed availability, spam filtering, data sovereignty

**Configuration:**

- Read: Public (anyone can subscribe)
- Write: Authenticated users only (prevents spam)
- Retention: 90 days for public events, indefinite for verification badges
- Backup: Daily snapshots to S3

### Public Relays (Fallback)

- `relay.damus.io` - High availability
- `nos.lol` - Popular, well-connected
- `relay.nostr.band` - Search/indexing support

**Strategy:**

- Publish to all relays simultaneously
- Subscribe from fastest responding relay
- Cache events in DynamoDB for instant UI updates

---

## Trust Graph Implementation

**Library:** Custom using NDK follow graph  
**Future:** Integrate [Grapevine](https://github.com/mplorentz/grapevine) for advanced trust scoring

### Simple 2-Degree Trust (MVP):

```javascript
// Get user's follows
const follows = await user.follows();

// Get follows of follows
const followsOfFollows = new Set();
for (const follow of follows) {
  const secondDegree = await follow.follows();
  secondDegree.forEach((f) => followsOfFollows.add(f.pubkey));
}

// Trust score
function getTrustScore(targetPubkey) {
  if (follows.has(targetPubkey)) return 1.0; // Direct follow
  if (followsOfFollows.has(targetPubkey)) return 0.5; // 2nd degree
  return 0.0; // Unknown
}
```

### Feed Filtering:

- L0 users: See only L2+ verified users
- L1+ users: See anyone with trust score > 0.3
- Configurable per user

---

## Data Privacy Considerations

### Location Privacy (Geohashing)

**Problem:** Don't expose exact addresses in public events  
**Solution:** Use geohash (variable precision location encoding)

**Example:**

```javascript
// Exact address: 123 Main St, Lawrence, MA 01841
const geohash = geohashEncode(42.707, -71.1631, 7); // "u4pruydqqvj"

// Published geohash (5 chars = ~5km radius)
const publicGeohash = geohash.substring(0, 5); // "u4pru"
```

**Precision Levels:**

- 4 chars: ~40km (city level)
- 5 chars: ~5km (neighborhood level)
- 6 chars: ~1.2km (few blocks)
- Full address: Shared only in NIP-17 encrypted DM

### Sensitive Data Never in Public Events:

- ❌ Full address
- ❌ Phone number
- ❌ Email
- ❌ Government IDs
- ❌ Payment info

**Only in encrypted DMs or custodial database (encrypted at rest).**

---

## Event Kinds Reference

| Kind  | NIP    | Purpose                   | Example                |
| ----- | ------ | ------------------------- | ---------------------- |
| 0     | NIP-01 | User metadata             | Profile info           |
| 1     | NIP-01 | Short text note           | Updates, announcements |
| 3     | NIP-02 | Contact list              | Follow graph           |
| 4     | NIP-04 | Encrypted DM (deprecated) | Legacy messages        |
| 8     | NIP-58 | Badge award               | Sentinel verification  |
| 14    | NIP-17 | Private message           | Coordination details   |
| 30009 | NIP-58 | Badge definition          | Trust badge types      |
| 30402 | NIP-99 | Classified listing        | Service requests       |

---

## Testing & Development

### Local Relay Setup (Optional):

```bash
# Install strfry
git clone https://github.com/hoytech/strfry
cd strfry && make

# Run locally
./strfry relay --config=strfry.conf
```

### Testing Tools:

- **rana:** CLI tool for generating Nostr keypairs
- **nostr-tools:** JavaScript library for manual event crafting
- **nostrudel:** Web client for debugging events
- **nostr.watch:** Monitor relay health

---

## Next Steps

1. **Week 1-2:** Implement NIP-99 service request publishing
2. **Week 3-4:** Add NIP-17 encrypted DM coordination
3. **Week 5-6:** Integrate NIP-07 browser extension support
4. **Week 7-8:** Deploy strfry relay with spam filtering
5. **Week 9-10:** Implement trust graph filtering

---

## Resources

- **Nostr Protocol:** [github.com/nostr-protocol/nostr](https://github.com/nostr-protocol/nostr)
- **NIPs Repository:** [github.com/nostr-protocol/nips](https://github.com/nostr-protocol/nips)
- **NDK Documentation:** [github.com/nostr-dev-kit/ndk](https://github.com/nostr-dev-kit/ndk)
- **strfry Relay:** [github.com/hoytech/strfry](https://github.com/hoytech/strfry)
