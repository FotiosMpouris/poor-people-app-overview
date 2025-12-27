# Technical Decisions - Why We Chose This Stack

**Purpose:** Document key technical decisions and rationale  
**Last Updated:** December 2025

---

## Frontend: Next.js 15 + React + TypeScript

### Why Next.js?

**Chosen Over:** Plain React (CRA), Vue.js, Svelte, Angular

**Reasons:**

1. **Server-Side Rendering (SSR):** Better SEO for landing pages, faster initial load
2. **App Router:** Modern file-based routing with layouts and nested routes
3. **API Routes:** Can handle backend logic without separate server (useful for Lambda proxying)
4. **TypeScript Support:** First-class, zero config
5. **Deployment Flexibility:** Works on Vercel, AWS Amplify, or self-hosted (EC2)
6. **Large Ecosystem:** Massive community, tons of Nostr-compatible libraries

**Tradeoffs:**

- ❌ Heavier than plain React (but benefits outweigh cost)
- ❌ Learning curve for App Router (new paradigm)
- ✅ Future-proof: Next.js 15 is cutting-edge, stable release

---

### Why TypeScript?

**Chosen Over:** Plain JavaScript, Flow

**Reasons:**

1. **Type Safety:** Catch Nostr event structure errors at compile time
2. **Better DX:** Auto-complete for NDK methods, fewer runtime bugs
3. **Documentation:** Types serve as inline docs for complex objects
4. **Refactoring:** Safe to change interfaces without breaking code

**Example:**

```typescript
// Nostr event type safety
interface NostrEvent {
  id: string;
  pubkey: string;
  created_at: number;
  kind: number;
  tags: string[][];
  content: string;
  sig: string;
}

// Compiler error if we miss a field
const event: NostrEvent = {
  id: "abc123",
  pubkey: "...",
  // Error: missing created_at, kind, tags, content, sig
};
```

**Tradeoffs:**

- ❌ Slower initial development (must define types)
- ✅ Faster debugging (errors caught before runtime)
- ✅ Easier onboarding (new devs see clear contracts)

---

### Why Tailwind CSS?

**Chosen Over:** CSS Modules, Styled Components, Bootstrap, Material UI

**Reasons:**

1. **Utility-First:** Rapid prototyping without leaving JSX
2. **No CSS File Bloat:** Purges unused styles in production
3. **Consistent Design:** Enforces design system (spacing, colors) through config
4. **Responsive:** Mobile-first breakpoints built-in (`md:`, `lg:`)
5. **Dark Mode:** First-class support (important for accessibility)

**Example:**

```jsx
<button className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
  Get Verified
</button>
```

**Tradeoffs:**

- ❌ Verbose className strings (mitigated with components)
- ✅ No context switching between files
- ✅ Tiny production bundle (~10kb)

---

## Nostr Layer: NDK (Nostr Development Kit)

### Why NDK?

**Chosen Over:** nostr-tools, nostr-relay-pool, custom implementation

**Reasons:**

1. **React Hooks:** `useNDK()`, `useSubscription()` integrate seamlessly
2. **Relay Management:** Auto-connects to best relays, handles failures
3. **Caching:** Reduces redundant relay queries
4. **TypeScript:** Full type definitions for events, filters, users
5. **Active Development:** Maintained by Pablo (Nostr core dev), frequent updates
6. **User Abstraction:** Easy profile fetching, follow lists, DM encryption

**Example:**

```typescript
import NDK, { NDKEvent } from "@nostr-dev-kit/ndk";

const ndk = new NDK({
  explicitRelayUrls: ["wss://relay.poorpeople.app"],
});
await ndk.connect();

// Subscribe to service requests
const filter = { kinds: [30402], "#t": ["ppa"] };
const sub = ndk.subscribe(filter);
sub.on("event", (event: NDKEvent) => {
  console.log("New request:", event.content);
});
```

**Tradeoffs:**

- ❌ Adds ~100kb to bundle (acceptable for features gained)
- ✅ Saves 100+ hours of custom relay logic
- ✅ Future-proof: NDK evolves with Nostr protocol

---

### Why Self-Hosted strfry Relay?

**Chosen Over:** Only public relays, custom relay, Nostr.wine

**Reasons:**

1. **Data Sovereignty:** We control what stays available
2. **Spam Filtering:** Can implement write restrictions (authenticated users only)
3. **Performance:** Geographic proximity to users (AWS us-east-1 for East Coast)
4. **Backup:** Direct access to relay database for disaster recovery
5. **Cost:** Free compute on EC2 (already paying for instance)

**strfry Specifically:**

- Written in C++ (extremely fast)
- Supports all NIPs we need
- Simple config file
- Active development, responsive maintainer

**Example Config (`strfry.conf`):**

```conf
relay {
  bind = "0.0.0.0"
  port = 7777

  info {
    name = "PPA Relay"
    description = "Poor People App coordination relay"
    pubkey = "..."
  }
}

events {
  maxEventSize = 50000
  rejectFutureSeconds = 900
}
```

**Tradeoffs:**

- ❌ Requires server maintenance (updates, monitoring)
- ✅ Full control over availability
- ✅ Can implement custom policies (e.g., NIP-13 PoW requirements)

---

## Backend: AWS Hybrid Architecture

### Why AWS Lambda (Not EC2-Only)?

**Chosen Over:** Monolithic EC2 server, Kubernetes, Cloud Run

**Reasons:**

1. **Cost Efficiency:** Pay only when functions execute (not 24/7 server costs)
2. **Scalability:** Auto-scales to handle spikes (e.g., viral request)
3. **Isolation:** Each function independent (bug in one doesn't crash others)
4. **Existing Infrastructure:** Already using Lambda for Mobilization Agent
5. **DynamoDB Integration:** Native event triggers (DynamoDB Streams → Lambda)

**Use Cases for Lambda:**

- Process incoming Nostr events from relay
- Send notifications (email, push)
- Background trust score calculations
- Scheduled jobs (daily backups, stale request cleanup)

**Example:**

```python
# Lambda triggered by DynamoDB Stream
def lambda_handler(event, context):
    for record in event['Records']:
        if record['eventName'] == 'INSERT':
            new_request = record['dynamodb']['NewImage']
            # Send notification to nearby users
            notify_potential_helpers(new_request)
```

**Tradeoffs:**

- ❌ Cold start latency (mitigated with provisioned concurrency for critical functions)
- ✅ Minimal ops overhead
- ✅ Integrated monitoring (CloudWatch)

---

### Why DynamoDB (Not PostgreSQL)?

**Chosen Over:** RDS PostgreSQL, MongoDB, Firestore

**Reasons:**

1. **Serverless:** No server to manage, auto-scales
2. **Geographic Queries:** Global Secondary Index on geohash for fast location filtering
3. **Low Latency:** Single-digit millisecond reads (critical for feed loading)
4. **Cost:** Free tier covers early usage, pay-per-request after
5. **Lambda Integration:** Native triggers for event processing

**Schema Design:**

```
Table: PPA-ServiceRequests
Primary Key: requestId (String)
Sort Key: created_at (Number)
GSI: geohash-index
  - Partition Key: geohash (String, 5 chars)
  - Sort Key: created_at (Number)
Attributes:
  - pubkey (requester's Nostr pubkey)
  - title, description, category
  - trust_level_required (L1, L2, L3)
  - status (open, matched, completed)
```

**Query Example:**

```javascript
// Get all open requests in Lawrence, MA area
const params = {
  TableName: "PPA-ServiceRequests",
  IndexName: "geohash-index",
  KeyConditionExpression: "geohash = :geo AND created_at > :time",
  FilterExpression: "status = :status",
  ExpressionAttributeValues: {
    ":geo": "u4pru", // Lawrence geohash
    ":time": Date.now() - 7 * 24 * 60 * 60 * 1000, // Last 7 days
    ":status": "open",
  },
};
const result = await dynamodb.query(params).promise();
```

**Tradeoffs:**

- ❌ No SQL joins (must denormalize or multiple queries)
- ✅ Predictable performance at scale
- ✅ No database administration

---

### Why S3 (For Backups & Static Assets)?

**Chosen Over:** EC2 disk, Cloudflare R2, Google Cloud Storage

**Reasons:**

1. **Durability:** 99.999999999% (11 nines) - data won't be lost
2. **Lifecycle Policies:** Auto-archive old backups to Glacier (cheap long-term storage)
3. **CDN Integration:** CloudFront caching for static assets (profile photos, badges)
4. **Versioning:** Keep multiple versions of critical files (relay database snapshots)
5. **Cost:** $0.023/GB/month (incredibly cheap)

**Use Cases:**

- Daily strfry database backups
- User profile photos
- Verification badge images
- Campaign logs (Mobilization Agent history)

**Tradeoffs:**

- ❌ Not a database (can't query directly)
- ✅ Perfect for write-once, read-many data
- ✅ Integrates with Lambda for automated backups

---

## Deployment Strategy

### Why EC2 + PM2 (For Now)?

**Chosen Over:** Vercel, AWS Amplify, Kubernetes, Docker Swarm

**Reasons:**

1. **Full Control:** Can run strfry relay + Next.js on same instance
2. **Cost:** Single t3.medium (~$30/month) handles early traffic
3. **Simplicity:** No orchestration complexity (yet)
4. **PM2 Process Management:** Auto-restart on crashes, log management
5. **Existing Experience:** Already running Hard-E this way

**PM2 Config:**

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: "ppa-frontend",
      script: "npm",
      args: "start",
      cwd: "/home/ec2-user/ppc-app",
      env: {
        NODE_ENV: "production",
        PORT: 3000,
      },
    },
    {
      name: "strfry-relay",
      script: "/usr/local/bin/strfry",
      args: "relay",
      cwd: "/home/ec2-user/strfry",
    },
  ],
};
```

**Future Migration Path:**

- **Phase 1 (Now):** EC2 + PM2
- **Phase 2 (1000+ users):** Frontend → Vercel, Relay stays on EC2
- **Phase 3 (10k+ users):** Multi-region EC2s with load balancer
- **Phase 4 (100k+ users):** Kubernetes for relay cluster

**Tradeoffs:**

- ❌ Manual scaling (must resize instance)
- ✅ Cheap and simple
- ✅ Easy to debug (SSH directly)

---

## Authentication Strategy

### Why Hybrid Custodial + Self-Custody?

**Chosen Over:** Only custodial, only self-custody, email-only (no Nostr keys)

**Reasons:**

1. **Accessibility:** Non-technical users need email/password option
2. **Sovereignty:** Power users can bring their own Nostr keys
3. **Portability:** Self-custody users can leave PPA anytime (keep their identity)
4. **Trust:** We don't want to "own" user identities long-term

**Custodial Flow:**

```
User signs up with email/password
→ PPA generates Nostr keypair
→ Private key encrypted with AWS KMS
→ Stored in DynamoDB
→ User can export keys later (encouraged)
```

**Self-Custody Flow:**

```
User clicks "Sign in with Nostr Extension"
→ Browser extension (Alby, nos2x) prompts for permission
→ PPA receives public key only
→ All event signing happens in extension
→ PPA never sees private key
```

**Key Management (Custodial):**

```python
import boto3
from cryptography.fernet import Fernet

kms = boto3.client('kms')

def encrypt_nsec(nsec: str, user_id: str) -> str:
    # Generate data key from KMS
    key = kms.generate_data_key(KeyId='alias/ppa-keys', KeySpec='AES_256')
    cipher = Fernet(key['Plaintext'])
    encrypted = cipher.encrypt(nsec.encode())

    # Store encrypted nsec + encrypted key
    return {
        'encrypted_nsec': encrypted,
        'encrypted_key': key['CiphertextBlob']
    }
```

**Tradeoffs:**

- ❌ Custodial users trust us with keys (mitigated by KMS encryption)
- ✅ Lowest barrier to entry
- ✅ Path to self-sovereignty (export feature)

---

## Development Tools

### Why VS Code (Not Cursor, Vim, IntelliJ)?

**Chosen Over:** JetBrains WebStorm, Sublime, Emacs

**Reasons:**

1. **Free & Open Source**
2. **Extensions:** ESLint, Prettier, Tailwind IntelliSense built-in
3. **Remote Development:** SSH into EC2 directly
4. **Git Integration:** Built-in, works great
5. **Debugging:** Chrome DevTools integration

**Key Extensions:**

- Tailwind CSS IntelliSense
- ESLint + Prettier
- TypeScript + JavaScript Grammar
- GitLens (commit history)
- AWS Toolkit (Lambda deployment)

---

### Why Git + GitHub (Not GitLab, Bitbucket)?

**Reasons:**

1. **Community:** Largest open-source community
2. **Actions:** Free CI/CD for public repos
3. **Visibility:** Easier to attract contributors
4. **Integration:** Works with Vercel, AWS, etc.

---

## Monitoring & Observability

### Why CloudWatch (For Now)?

**Chosen Over:** Datadog, New Relic, Sentry

**Reasons:**

1. **Native AWS Integration:** Lambda logs auto-sent
2. **Cost:** Free tier generous (10 custom metrics, 5GB logs)
3. **Dashboards:** Visual monitoring without third-party
4. **Alarms:** SNS notifications on errors/latency spikes

**Future:** Add Sentry for frontend error tracking (user-facing bugs)

---

## Open Questions & Future Decisions

### 1. Mobile Apps

**Options:**

- React Native (reuse React skills)
- Progressive Web App (no app store approval)
- Flutter (better performance, learning curve)

**Decision:** PWA first, React Native if usage justifies

---

### 2. Payment Integration

**Options:**

- Lightning Network (native Nostr integration)
- Stripe (fiat, easier adoption)
- Hybrid (both)

**Decision:** Lightning first (aligns with Nostr ecosystem), Stripe if users demand

---

### 3. AI Features

**Options:**

- GPT-4 for matching suggestions ("This helper seems like a good fit because...")
- Sentiment analysis on DMs (flag concerning language)
- Auto-categorization of requests

**Decision:** Avoid AI initially (focus on human trust), revisit if clear use case

---

### 4. Analytics

**Options:**

- Google Analytics (free, privacy concerns)
- Plausible (privacy-first, $9/month)
- Self-hosted Matomo (free, requires maintenance)

**Decision:** Plausible (aligns with privacy-first values)

---

## Summary: The Stack

```
Frontend:
  ├─ Next.js 15 (App Router)
  ├─ React 18 + TypeScript
  ├─ Tailwind CSS
  └─ NDK (Nostr integration)

Backend:
  ├─ AWS Lambda (event processing)
  ├─ DynamoDB (caching layer)
  ├─ S3 (backups, static assets)
  └─ CloudWatch (monitoring)

Nostr:
  ├─ Self-hosted strfry relay
  ├─ Public relays (damus.io, nos.lol)
  └─ NIPs: 01, 07, 17, 58, 99, 13

Deployment:
  ├─ EC2 t3.medium (frontend + relay)
  ├─ PM2 (process management)
  ├─ Nginx (reverse proxy)
  └─ Route53 (DNS)

Dev Tools:
  ├─ VS Code + Extensions
  ├─ Git + GitHub
  ├─ AWS CLI
  └─ Cursor (AI assistance)
```

---

## Principles Guiding Decisions

1. **Simplicity Over Perfection:** Ship MVP, iterate based on usage
2. **Open Standards:** Nostr > proprietary protocols
3. **User Sovereignty:** Self-custody option always available
4. **Privacy First:** Minimal data collection, encryption by default
5. **Cost Efficiency:** Bootstrap-friendly, scale when needed
6. **Developer Experience:** Tools should accelerate, not frustrate

---

**Last Updated:** December 26, 2025  
**Next Review:** After MVP launch (Q1 2025)
