# User Flows - Step-by-Step Interaction Patterns

**Purpose:** Document every user journey through the PPA platform  
**Last Updated:** December 2025

---

## Flow 1: New User Onboarding

### Entry Point: Landing Page Visit

**Step 1: Landing Page**

```
User arrives at poorpeople.app
Sees hero message: "Your neighbors need help. You can help them."
Call to action: "Get Started" button
```

**Step 2: Account Creation**

```
Option A: Email/Password (Custodial)
- Enter email, password, confirm password
- Enter zip code (for geo-filtering)
- Agree to Terms of Service
- Click "Create Account"
- PPA generates Nostr keypair (stored encrypted)
- Email verification sent

Option B: Browser Extension (Self-Custody)
- Click "Sign in with Nostr Extension"
- Alby/nos2x prompts for permission
- User approves, pubkey retrieved
- Enter zip code only
- No password needed (extension manages keys)
```

**Step 3: Profile Setup**

```
Upload profile photo (optional)
Enter display name
Write short bio (50-150 chars)
Select interests/skills (checkboxes)
- Tech help
- Yard work
- Tutoring
- Elder care
- etc.
```

**Step 4: Trust Level Explanation**

```
Modal appears: "Welcome to PPA!"
Explains trust ladder (L0 → L1 → L2 → L3)
Current status: Level 0 (New User)
Options to advance:
- Build connections (invite friends)
- Schedule verification appointment
User clicks "Got it" → Enters main app
```

**Step 5: Initial Feed**

```
Shows 10-15 sample service requests from L2+ users
Geofenced to user's area (5km radius)
Banner at top: "You're at Level 0. Verify your identity to unlock full features."
Click banner → Verification scheduling page
```

---

## Flow 2: Posting a Service Request

### Entry Point: Dashboard, "Need Help" Button

**Step 1: Choose Category**

```
Grid of icons:
- Groceries & Errands
- Tech Help
- Yard Work & Cleaning
- Tutoring & Education
- Elder Check-ins
- Transportation
- Other (custom)
User clicks one → Proceeds to form
```

**Step 2: Fill Request Form**

```
Title: (One-line summary, max 60 chars)
Description: (Detailed explanation, max 500 chars)
Location:
  - Auto-filled from profile zip code
  - User can adjust radius (1km, 5km, 10km)
  - Exact address added later in DM
Timeframe:
  - Date picker (start date)
  - Time estimate (30 min, 1 hour, 2 hours, half day, full day)
Expected Outcome: (What does "done" look like?)
Trust Level Required:
  - Any verified user (L1+)
  - Hub verified (L2+)
  - Sentinel verified (L3 only)
```

**Step 3: Preview & Publish**

```
Shows preview of public listing:
  "Need help with [Title]"
  "In [Geohash neighborhood]"
  "Estimated time: [X hours]"
  "Seeking [L2+ verified helpers]"
Warning: "Your exact address will only be shared after you accept a helper."
Click "Publish" → Event signed & broadcast to Nostr relays
```

**Step 4: Confirmation**

```
Success modal: "Your request is live!"
Shows:
  - Link to view listing
  - Estimated response time (based on area activity)
  - "You'll get notified when someone offers to help"
Returns to dashboard with active listing visible
```

---

## Flow 3: Offering to Help (Responding to Request)

### Entry Point: Browse Feed, See Interesting Request

**Step 1: View Request Details**

```
User clicks on service request card in feed
Modal expands with full details:
  - Title & description
  - Posted by: [Name + verification badge]
  - Location: [Neighborhood, geohash]
  - Time estimate: [X hours]
  - Expected outcome
  - Posted: [X hours ago]
"I Can Help" button at bottom
```

**Step 2: Trust Check (App-Side)**

```
IF user is L0:
  → "You must verify your identity to offer services."
  → Button changes to "Get Verified"

IF trust level insufficient:
  → "This request requires L2+ verification."
  → Shows progress: "You're L1. Schedule hub verification."

IF trust level sufficient:
  → Proceed to commit screen
```

**Step 3: Commitment Screen**

```
Confirm availability:
  - "I can do this on [date picker]"
  - "I can commit [time estimate]"
  - "Any questions or special considerations?"
    (Optional message to requester)
Click "Send Offer" → Initiates NIP-17 encrypted DM
```

**Step 4: Waiting for Acceptance**

```
Status changes to "Offer Sent"
Notification sent to requester
User sees in "My Activity":
  - Request title
  - Status: Pending
  - Estimated response time: 24 hours
If accepted → Notification + DM thread opens
If declined → Notification with optional reason
```

---

## Flow 4: Private Coordination (After Match)

### Entry Point: Requester Accepts Helper's Offer

**Step 1: Acceptance Notification**

```
Requester reviews offers (may have multiple)
Selects best fit:
  - Checks helper's profile
  - Reviews trust score/badges
  - Sees completion history
Clicks "Accept [Helper Name]'s Offer"
Encrypted DM thread created automatically
```

**Step 2: DM Exchange (Auto-Prompted)**

```
App pre-fills first message from requester:
"Hi [Helper]! Thanks for offering to help. Here are the details:

📍 Exact Address: [User types]
📞 Phone: [User types]
🗓️ Preferred Date/Time: [User types]
💬 Additional notes: [User types]"

Helper receives notification → Responds
Thread continues like normal chat
```

**Step 3: Final Confirmation**

```
Both parties confirm in DM:
Requester: "See you [Date] at [Time]!"
Helper: "Confirmed! I'll be there."

App prompts both:
"Mark this task as scheduled?"
→ Status updates from "Matched" to "Scheduled"
→ Calendar reminder added (if user enabled)
```

**Step 4: Check-In (Day Of)**

```
App sends reminder notifications:
- 1 day before: "Your appointment with [Name] is tomorrow"
- 1 hour before: "Your appointment starts soon"
- On completion: "Mark task as complete?"
```

---

## Flow 5: Completion & Rating

### Entry Point: Task Finished, Both Parties on Site

**Step 1: Helper Marks Complete**

```
Helper clicks "Task Complete" button in app
Confirmation prompt:
  "Did you complete the task as described?"
  [Yes] [No, there was an issue]
If Yes:
  → Creates completion event (kind TBD)
  → Notification sent to requester
If No:
  → Prompts for explanation
  → Escalates to Sentinel review
```

**Step 2: Requester Confirms**

```
Requester receives notification:
"[Helper] marked your task complete. Confirm?"
Options:
  ✅ "Yes, task completed successfully"
  ⚠️ "Partially completed"
  ❌ "Task not completed"

If Yes:
  → Both parties earn reputation points
  → Completion event published to Nostr
  → Prompt for optional rating

If Partially/Not:
  → Requires explanation
  → Escalated to Sentinel
```

**Step 3: Optional Rating (Future Feature)**

```
5-star scale:
⭐⭐⭐⭐⭐ "Excellent"
⭐⭐⭐⭐ "Good"
⭐⭐⭐ "Okay"
⭐⭐ "Below expectations" (requires comment)
⭐ "Poor" (requires comment + Sentinel review)

Optional comment (max 280 chars)
Privacy:
  - 4-5 stars: Public
  - 1-3 stars: Visible to L2+ users only
```

**Step 4: Reputation Update**

```
Both profiles updated:
Helper:
  - Total completions: +1
  - Average rating: recalculated
  - Badge earned (e.g., "10 Completions")

Requester:
  - Requests fulfilled: +1
  - Active participant badge
```

---

## Flow 6: Hub Verification (L0 → L2)

### Entry Point: User Clicks "Get Verified" Banner

**Step 1: Choose Hub Location**

```
Map shows nearby hub locations:
- Lawrence Senior Center (0.5 mi)
- Methuen Library (2.3 mi)
Each shows:
  - Next available appointment
  - Address
  - Sentinel on duty
User selects → Proceeds to scheduling
```

**Step 2: Schedule Appointment**

```
Calendar view shows available slots:
- Day/time options
- 30-minute blocks
- Up to 2 weeks in advance
User picks slot → Confirmation screen
```

**Step 3: Appointment Confirmed**

```
Email + in-app notification sent:
"Verification appointment scheduled:
  📅 Date: [Date]
  🕐 Time: [Time]
  📍 Location: [Address]
  👤 Sentinel: [Name]

What to bring:
  ✅ Government-issued photo ID
  ✅ Smartphone (to receive verification)

See you there!"
```

**Step 4: Day of Appointment**

```
Reminder notification 1 hour before
User arrives at hub
Sentinel greets, reviews ID
Brief interview (~5 min)
Sentinel approves → Publishes L2 badge immediately
User sees notification: "You're now Level 2!"
Returns to app → Full features unlocked
```

---

## Flow 7: Dispute Resolution

### Entry Point: User Reports Concern

**Step 1: Submit Report**

```
From DM thread or profile:
Click "Report Issue"
Select category:
  - Safety concern
  - Task not completed as agreed
  - Inappropriate behavior
  - Scam/fraud attempt
  - Other
Provide details (required, max 500 chars)
Attach evidence (screenshots, photos - optional)
Click "Submit Report"
```

**Step 2: Sentinel Review**

```
Escalation Sentinel notified immediately (if safety)
Otherwise: Reviewed within 24 hours
Sentinel sees:
  - Report details
  - Both users' profiles
  - Completion history
  - DM thread (if consent given)
Makes decision: No Action / Warning / Suspension / Ban
```

**Step 3: User Notification**

```
Both parties notified of decision:
"Your report about [User] has been reviewed.
Decision: [Warning issued]
Details: [Brief explanation]
If you disagree, you can appeal within 7 days."

Accused party:
"A report was filed against you.
Decision: [Warning - no account action]
Details: [What happened]
Future violations may result in suspension."
```

**Step 4: Appeals (If Requested)**

```
User clicks "Appeal Decision"
Writes explanation (max 500 chars)
Lead Sentinel reviews appeal
Final decision made (no further appeals)
```

---

## Flow 8: Building Social Graph (L0 → L1)

### Entry Point: New User Wants to Avoid In-Person Verification

**Step 1: Invite Friends**

```
Dashboard shows: "Invite friends to build your network"
Options:
  - Share invite link (SMS, email, social media)
  - Connect Nostr following list (if using extension)
  - Search for existing users by name
Goal: Get 10+ connections with L1+ users
```

**Step 2: Accept Follow Requests**

```
Notifications when someone follows you:
"[Name] started following you"
Options:
  - Follow back (mutual connection)
  - Ignore
Mutual connections count toward L1 threshold
```

**Step 3: Automatic Upgrade**

```
When threshold reached (10 mutuals OR trust score >0.3):
Notification: "You've reached Level 1!"
New capabilities unlocked:
  - Offer services at public locations
  - See broader feed
  - Message L2+ users
```

---

## Edge Cases & Error States

### 1. No One Responds to Request

```
After 48 hours, app prompts:
"No responses yet. Options:
  - Extend deadline
  - Broaden radius
  - Increase visibility (boost with PPC tokens - future)
  - Post in nearby communities"
```

### 2. Helper No-Shows

```
Scheduled time passes, task not marked complete
App asks requester: "Did [Helper] show up?"
If No:
  → Helper flagged (3 no-shows = temporary suspension)
  → Requester can repost request
  → Incident logged for Sentinel review
```

### 3. Payment Disputes (Future)

```
When Lightning payments integrated:
Escrow released only after both confirm completion
Disputed payments held 7 days:
  - Parties negotiate
  - If unresolved → Sentinel arbitration
```

---

## Mobile vs. Desktop Considerations

**Mobile (Future Native App):**

- Push notifications for real-time updates
- GPS location for geofencing
- Camera for ID upload
- Biometric login

**Desktop (Current Web MVP):**

- Email notifications
- Manual zip code entry
- File upload for photos
- Password login

---

## Next Steps

- [ ] Build wireframes for each flow
- [ ] User testing with mockups (5-10 pilot users)
- [ ] Refine based on feedback
- [ ] Implement in Next.js + NDK
