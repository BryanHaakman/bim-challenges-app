# BIM Challenges — Product Requirements Document (MVP)

**Status:** Draft for team review
**Target:** [name].boisinmotion.com (subdomain, Vercel)

---

## 1. Problem & Opportunity

### Opportunity
The December challenge (streaks, dares, workouts, self-organized in group chat + spreadsheet) proved the concept — engagement was way higher than expected. That's a real signal, not just a hunch: BIM's circle already showed it will show up, log proof, and put money on the line for a shared challenge without much prompting. The app is about capturing that demand with something purpose-built instead of duct tape.

### Pain Points (from December)
- **No single source of truth for verification** — proof lived scattered across group chat messages, easy to lose track of, no reliable way to look back and confirm who actually did what
- **Money/stakes were a headache** — manual spreadsheet math for who owes what, manual e-transfer collection and chasing people down
- **No persistent record** — once the group chat moved on, streaks/results/history weren't preserved anywhere durable
- **Everything was manually refereed** — no automated reminders, no structured way to raise or resolve a dispute, the organizer had to manually referee every edge case

### Value Proposition
The app replaces the spreadsheet + group chat with structure — verified proof, computed leaderboards, automated settlement — without killing the scrappy, social vibe that made December work. The feed and social layer are core, not an afterthought: this should feel like the thing the group already does, just with less friction and fewer arguments over who owes who.

### ICP (Ideal Participant Profile)
- **Who**: BIM's existing social circle — people who already know each other and already ran the December challenge informally. Not a general fitness-app audience, not a cold-acquisition product.
- **Profile**: recreational endurance athletes / fitness enthusiasts, comfortable putting real money on the line with friends, already on Strava or willing to log manually
- **Platform**: mobile-first — submitting proof, checking the leaderboard, and reacting to the feed will mostly happen on a phone, not desktop
- *(TBD — confirm with team: expected group/challenge size, and whether this stays scoped to one friend circle or there's near-term appetite to widen within BIM's broader following)*

**Single-tenant.** No multi-group support, no friends/follow system, no DMs. This is BIM's app for BIM's circle. Multi-group is a future direction but not in scope.

**Long-term directions (not MVP, but shape the data model):**
- Group training tied to a specific race (e.g. a group training together for a marathon) — would need training-plan adherence tracking (weekly mileage targets, etc.), parked for later
- AI/computer-vision proof verification (e.g. counting reps from video)

---

## 2. MVP Scope

### Core user stories
1. Create & join a challenge
2. Submit proof
3. Peer verify / dispute proof
4. Track streak / leaderboard standing
5. Settle up — computed ledger at challenge close
6. Discover challenges (browse by status, join via link or from discovery)
7. View a profile — stats, badges, history, click-through to challenge someone directly
8. 1-on-1 direct challenge ("duel") between two users
9. Team-based challenges (teams compete against teams)
10. Onboarding challenge — connect Strava + log one activity in week one, earn first badge

### Out of scope for MVP
- Multi-group / multi-tenant support
- Native mobile app (mobile-responsive web only)
- Push notifications (email only)
- Friends/follow system, direct messaging
- Badge rules/catalog beyond the one onboarding badge (schema should support more, logic is phase 2)
- AI/computer-vision proof verification
- Training-plan adherence tracking for race-training groups
- Content moderation tooling beyond manual organizer removal (trusted friend-group assumed)

---

## 3. Users & Roles

| Role | Description |
|---|---|
| **Organizer** | Any user can create and manage a challenge through to completion. No special account tier required. |
| **Co-admin** | Organizer can whitelist other participants as co-admins to help manage the challenge and resolve disputes |
| **Participant** | Joins a challenge, submits proof, verifies/disputes others' proof |

**Build team:** Bryan builds (vibe coding). Andrew gates technical structure/architecture approval. Tyler owns testing. Branding and MVP feature scope are agreed across all three.

No public/guest browsing beyond discovery previews — signup is required to join a challenge, not required to preview a challenge via invite link.

---

## 4. Core User Stories (detailed)

### 4.1 Create & Join a Challenge
- Organizer sets: name, description, **mode** (head-to-head / collaborative / custom-per-person / team), goal type (streak, distance, time, dare), start/end date, **timezone** (defaults to organizer's browser timezone — used for all streak resets and deadline calculations for this challenge), proof rules, verification mode, stakes (optional), payout rule (if staked), visibility (public/private), team structure (if team mode)
- Any user can create a challenge — no gating
- Join via: direct invite link (primary acquisition path — preview challenge details before any signup prompt), join code, or discovery tab
- New-user signup is deferred until they actually initiate joining, not shown upfront
- **Private challenges** are not listed in discovery but remain fully joinable via invite link or join code. A non-participant landing on a private challenge's invite link sees the preview (name, description, organizer, dates) and can sign up to join — same flow as public, just not discoverable.

### 4.2 Challenge Modes
Four shapes, in priority order for MVP:

- **Head-to-head**: everyone shares the identical goal, ranked against each other. Ties are allowed and left as ties, no tiebreaker.
- **Collaborative**: group works toward one shared target. Leaderboard shows both the overall shared progress bar AND a per-person contribution breakdown underneath.
- **Custom-per-person**: each participant sets their own goal (e.g. one person runs 5k/day, another attempts an Ironman-distance total, another goes for 100k in a day) — same mechanic that ran in December. Scored as % progress toward each person's own target.
- **Team**: multiple people grouped into a team, competing against other teams (e.g. 2v2). Team score aggregates member proof. Teams are organizer-assigned at challenge creation — no self-select or draft for MVP. Team size is set by the organizer and must be consistent across all teams in a challenge (no uneven teams). If a team member withdraws mid-challenge, their historical proof contributions remain counted toward the team's score; the team continues with fewer active members.

**Goal type × mode compatibility:**

| Goal type | Head-to-head | Collaborative | Custom | Team |
|---|---|---|---|---|
| **Streak** | Ranked by current streak (longest wins ties) | Sum of all members' active streak days | % = days_active / personal_target_days | Sum of team members' active streak days |
| **Distance** | Ranked by total distance logged | Group distance total | % = distance_logged / personal_target | Team distance total |
| **Time** | Ranked by total time logged | Group time total | % = time_logged / personal_target | Team time total |
| **Dare** | Binary — completed or not; rank by completion count if multi-dare | Group completion rate | % = 0% or 100% (dare is binary; valid but degenerate — leaderboard shows who did/didn't) | Team completion rate |

### 4.3 Submit Proof
- Photo, video, text note, or auto-pulled Strava activity (OAuth + webhook)
- Strava supports **retroactive backfill** — activities from before someone connected Strava can still count if they fall within the challenge window. Backfill is triggered once at connect time; the server fetches the user's Strava activity history and matches activities against active challenges by date.
- Strava-sourced proof auto-verifies (skips peer verification) unless flagged
- **Strava deduplication**: Strava `activity_id` is the idempotent key. If a Strava activity is submitted manually AND later pulled via webhook (or vice versa), the server deduplicates on `strava_activity_id` — only one proof record is kept. Manual submission wins if it already has a verification decision; otherwise the webhook record is canonical.
- **Strava activity edits**: if Strava sends an update webhook for an activity already used as proof, the server re-checks the updated activity against the challenge rules. If the edit makes it non-qualifying (e.g., distance drops below threshold), the proof status reverts to `pending` and the organizer is notified. If it still qualifies, no status change.
- **Cross-challenge deduplication**: the same Strava activity can count as proof in multiple simultaneous challenges — there is no restriction. Each challenge is independent.
- **Proof submissions are per-person** — one proof per submission event. Bulk upload is out of scope for MVP.

### 4.4 Peer Verify / Dispute
- Verification mode is an **organizer-configurable setting per challenge**:
  - **Single approval** — one peer approval marks the proof verified
  - **Auto-approve-unless-challenged** (default recommendation) — lowest friction, only creates work on actual disputes
- **Auto-approval timeout**: in `auto_unless_challenged` mode, a proof submission that receives no dispute within **48 hours** of submission is automatically marked `verified`. The 48-hour window starts at submission time.
- A disputed submission is **paused** — does not count toward the leaderboard until resolved. Resolved items are given the original submission timestamp (not the resolution timestamp) so standings reflect when the work actually happened.
- Resolution authority: organizer by default, or any co-admin they've whitelisted for that challenge
- **Challenge close with open disputes**: if the challenge end date passes while disputes remain unresolved, the challenge enters a `settling` state. The organizer has **7 days** to resolve all open disputes. After 7 days, any unresolved dispute is **auto-resolved in the submitter's favor** and the challenge force-closes to settlement.
- **Majority vote** mode is in the schema but not exposed in the MVP UI. Quorum logic for variable participant counts is deferred to phase 2. Only `single` and `auto_unless_challenged` are selectable at challenge creation.

### 4.5 Streak / Leaderboard
- Server-computed and cached per challenge
- Head-to-head: ranked by raw progress toward the goal
- Collaborative: group total + individual contribution breakdown
- Custom: % progress toward each person's own target
- Team: aggregated team score, ranked against other teams
- **Streak reset timezone**: all streak reset windows (e.g. "must submit once per calendar day") use the **challenge's timezone**, set by the organizer at creation. A participant in Vancouver and one in Toronto are both on the challenge's clock — no per-participant timezone adjustment.

### 4.6 Settle Up
- **Stripe handles stake collection and payout** — participants pay their stake via Stripe Checkout at join time; funds are held and disbursed at challenge close via Stripe Connect
- **Stripe Connect onboarding** is prompted **when a user joins a staked challenge**, not at challenge close. This gives users time to complete KYC before any payout is triggered. A user who hasn't completed Connect onboarding by the time payouts are computed will have their payout held and re-attempted once onboarding completes (up to 30 days, after which the organizer is notified to handle manually).
- **Payout rule is organizer-configurable per challenge** (winner-take-all, split among all who completed their goal, or other) — not a single fixed app-wide rule
- On challenge close, the server computes the settlement ledger and triggers Stripe payouts to winners automatically
- **Zero-completers default (confirmed):** if a staked challenge closes with no participants having completed their goal, the challenge's stakes are **voided** — each participant's Stripe charge is refunded in full (pro-rata, everyone gets their stake back), rather than settling to a payout. No `payout_rule` override path is needed for this case since the team confirmed the refund default as-is.
- **No refunds** if a participant drops out or goes quiet — stake stays in the pot by default
- **Exception**: organizer can manually trigger a Stripe refund for legitimate cases (injury, group-agreed fairness) — separate override from the default no-refund rule
- Two stake models: peer-funded (participants pay into their own pot via Stripe) and org-sponsored (BIM or sponsor funds a prize pool) — Stripe Connect handles disbursement for both. Prizes could also be physical goods (gear, nutrition, race entries) with manual fulfillment.

### 4.7 Discovery
Discovery tab reflects challenge lifecycle status, filterable by:
- **Upcoming** — not started yet, joinable or request-to-join
- **Active, open** — in progress, still accepting new joiners
- **Active, closed** — in progress, not accepting new joiners (spectate-only view for non-participants — they can see the leaderboard and feed but cannot submit proof or join)
- **Closed/archived** — disappears from discovery, still viewable from a participant's profile/history

**Private challenges do not appear in discovery** regardless of status. They are only accessible via direct invite link or join code.

Also supports creating a new challenge directly from this tab.

### 4.8 Profile
Includes: profile picture, name, badges, history of completed challenges, current active challenges. Clickable from anywhere a name/avatar appears (leaderboard, feed) — opens the profile and offers a "challenge this person" action, feeding into the 1-on-1 duel flow.

### 4.9 1-on-1 Duels
Direct challenge between two users, using the same proof/verification/leaderboard system as group challenges, just scoped to two participants. Discoverable from any profile view.

### 4.10 Onboarding
First-run flow on signup: connect Strava + complete one logged Strava activity within the first week → awards first badge. Any Strava activity type counts (run, ride, swim, workout, etc.) with no minimum distance or duration.

**Strava is optional, not mandatory.** Users who skip Strava connection can still join and participate in challenges using photo/video/text proof. They simply won't earn the onboarding badge and won't have Strava auto-verification available. The onboarding screen surfaces the Strava connection prominently but includes a clear "skip for now" path.

### 4.11 In-Challenge Social Feed
The feed is visible to all challenge participants (and spectators for active-closed challenges). It is scoped per challenge — there is no global cross-challenge feed for MVP.

**What appears in the feed (in reverse chronological order):**
- Proof submissions — photo thumbnail or activity summary card, with submitter name/avatar
- Verification events — "X verified Y's proof" or "X disputed Y's proof" (disputes visible to all participants, not just the organizer)
- Milestone hits — new personal streak record, goal completion, first submission of the challenge
- Challenge state changes — challenge started, "X days remaining" reminders (daily at 3 days out), challenge closed

**Reactions:** participants can react to any feed item with a fixed emoji set (e.g., 🔥 💪 👀 😬). No comments for MVP — reactions only.

---

## 5. Feature Set (MVP)

| Feature | Priority | Notes |
|---|---|---|
| Auth (email or Google) | P0 | Supabase Auth |
| User profiles (photo, name, badges, history) | P0 | |
| Challenge creation (head-to-head + collaborative) | P0 | Custom-per-person and team can slip if timeline is tight |
| Join via link/code/discovery | P0 | |
| Discovery tab with lifecycle filtering | P0 | |
| Proof submission (photo/video/text) | P0 | Supabase Storage |
| Strava OAuth + webhook + retroactive backfill | P0 | Reuse existing BIM Client ID |
| Peer verification/dispute (single + auto-unless-challenged) | P0 | Majority vote is phase 2 |
| Streak/leaderboard computation (per mode) | P0 | Server-authoritative |
| In-challenge social feed with emoji reactions | P0 | Per-challenge, no global feed |
| Stripe Checkout — stake collection at join time | P0 | Stripe Connect for disbursement |
| Ledger/settlement calculation w/ configurable payout rule | P0 | Server-triggered Stripe payout on close |
| Withdrawal + no-refund default + manual Stripe refund override | P0 | |
| Zero-completers auto-refund default | P0 | |
| 1-on-1 duel challenges | P0 | Same engine, 2-participant scope |
| Onboarding challenge + first badge | P0 | Strava optional; non-Strava users skip badge |
| Custom-per-person goal mode | P1 | Can follow head-to-head/collaborative if needed |
| Team-based challenges (2v2 etc.) | P1 | Stretch goal — first thing to cut if timeline slips |
| Email notifications (dispute raised, challenge closing, results) | P1 | No push for MVP |
| Co-admin whitelist per challenge | P1 | |
| Majority vote verification mode | P2 | Quorum logic deferred |
| Badge catalog beyond onboarding badge | P2 | Schema-ready, rules TBD |
| AI/computer-vision proof verification | P2 | Phase 2/3 — design proof storage (esp. video) to not preclude this later |
| Race-training group mode (adherence tracking) | P2 | Explicitly parked, noted for future |
| Org-sponsored prize pools | P2 | Ledger model supports it; real build TBD |

---

## 6. Branding & Design

Branding and visual design aren't decided yet and need a deliberate pass before UI build starts in earnest — a real design consideration, not something to sort out ad hoc mid-Phase-1.

**Process:**
1. **Discuss & gather ideas** — align on tone (scrappy/social vs. polished/athletic), reference apps/inspiration, and whether to inherit existing BIM brand assets (logo, colors, fonts) or depart from them
2. **Draft initial directions** — build a small set of visual directions (color palette, typography, component style, a few key screens) using Claude Design, so the team is reacting to something concrete instead of describing preferences in the abstract
3. **Team approval gate** — the team reviews the drafted directions and signs off on one before it's built into the app. This is a checkpoint, not a formality — building UI against an unapproved direction risks throwaway work
4. **Lock initial design system** — once approved, the chosen direction becomes the baseline design system (colors, type scale, spacing, core components) that Phase 1 UI work builds against. Refinement continues after, but the direction itself shouldn't flip mid-build

**Scope for MVP:** enough of a design system to build Phase 1 consistently (buttons, cards, leaderboard rows, feed items, proof cards) — not a full brand guidelines document or component library up front.

*(See Open Questions #11 for timing relative to Phase 1 build start.)*

---

## 7. Technical Architecture

**Stack:** Next.js (App Router) + Supabase (Postgres, Auth, Storage) + Vercel + Sentry + PostHog + Resend + React Email.

```
[name].boisinmotion.com (Vercel — deployment)
  ├─ Next.js frontend + server actions
  ├─ Supabase Auth (email + Google OAuth)
  ├─ Supabase Postgres (challenges, proofs, verifications, ledger, teams)
  ├─ Supabase Storage (proof photos/video)
  ├─ Strava OAuth + webhook subscription
  ├─ Stripe (Checkout for stake collection, Connect for payouts, webhooks for payment events)
  ├─ Sentry (error tracking + performance monitoring)
  ├─ PostHog (product analytics, session replay)
  ├─ Vercel (deployment)
  ├─ Resend (transactional email delivery — disputes, reminders, results)
  └─ React Email (email templates as React components)
```

### Data model (MVP, updated)

```
users
  id, email, display_name, avatar_url, strava_athlete_id, strava_tokens (encrypted),
  stripe_account_id (Connect account for receiving payouts),
  stripe_onboarding_complete (bool)

challenges
  id, organizer_id, name, description,
  mode (head_to_head | collaborative | custom | team),
  goal_type (streak | distance | time | dare),
  start_date, end_date, timezone (IANA tz string, e.g. "America/Toronto"),
  proof_rules (jsonb),
  verification_mode (single | auto_unless_challenged | majority),
  verification_timeout_hours (int, default 48 — applies to auto_unless_challenged),
  stake_amount, payout_rule (jsonb),
  visibility (public | private), status (upcoming | active_open | active_closed | settling | closed),
  join_code

challenge_admins
  id, challenge_id, user_id   -- co-admin whitelist

challenge_participants
  id, challenge_id, user_id, team_id (nullable), joined_at,
  status (active | withdrawn | withdrawn_refunded), personal_goal (jsonb, for custom mode)

teams
  id, challenge_id, name

proofs
  id, challenge_id, participant_id, submitted_at,
  type (photo | video | text | strava), content_url, strava_activity_id,
  status (pending | verified | disputed | rejected)

verifications
  id, proof_id, verifier_id, vote (approve | dispute), reason, created_at

feed_events
  id, challenge_id, event_type (proof_submitted | proof_verified | proof_disputed |
    milestone_hit | challenge_started | challenge_closing | challenge_closed),
  actor_id (nullable), subject_id (nullable, e.g. proof_id), metadata (jsonb), created_at

feed_reactions
  id, feed_event_id, user_id, emoji, created_at

standings (denormalized, recomputed on verification)
  id, challenge_id, participant_id, team_id (nullable),
  current_streak, longest_streak, completion_pct, contribution_value, rank

ledger
  id, challenge_id, from_user_id, to_user_id, amount,
  stripe_payment_intent_id, stripe_transfer_id,
  settled (bool), settled_at

badges / user_badges
  id, name, criteria (jsonb) / user_id, badge_id, earned_at
```

### Key architectural notes
- **Server-authoritative math everywhere** — streaks, standings, ledger, never trust client values
- **All time calculations use the challenge's `timezone` field** — streak resets, deadline enforcement, and "days remaining" calculations all operate in challenge-local time, not UTC and not participant-local time
- **Strava via webhook subscription**, not polling — matches activity to challenge rules server-side before auto-verifying; must support backfilled/past-dated activities within the challenge window; idempotent on `strava_activity_id`
- **Strava webhook handles three event types**: `activity.create`, `activity.update`, `activity.delete` — all three must be handled. Update re-evaluates qualification; delete reverts proof to `rejected` and triggers standings recompute
- **Disputed proof pauses standings recompute** for that participant until resolved
- **48-hour auto-approval** — `pending` proofs in `auto_unless_challenged` challenges older than `verification_timeout_hours` are marked `verified` automatically
- **`settling` status** — challenge enters this state when end_date passes with open disputes. After 7 days, any unresolved disputes are auto-resolved in the submitter's favor and the challenge advances to `closed`
- **Team mode aggregates member-level standings into a team-level rank** — build the individual layer first, team is a rollup on top
- Proof storage should assume video may later feed a CV pipeline — store original resolution and format, no transcoding that discards quality
- **Stripe Checkout** collects stakes at join time; **Stripe Connect** handles payout disbursement to winners — Stripe webhooks confirm payment success before participant is marked active in a staked challenge
- **Stripe Connect onboarding** is prompted at join time for staked challenges, not at close — gives users time to complete KYC. Unfinished onboarding holds the payout for up to 30 days before escalating to manual organizer handling
- **Content moderation**: no automated tooling for MVP. Organizers can remove proof submissions and ban participants from their challenge. Inappropriate content flagged to the broader BIM circle is handled out-of-band. Re-evaluate if the platform opens beyond the current trusted group.

---

## 8. Open Questions for the Team (Andrew, Tyler, Tomek)

1. **App name** Shortlist: Challenge, App, Grind, Pact, Grit, Stakes, Ante, Reps, Commit.

2. **Team mode: MVP or cut?** Flagged P1/stretch — worth an explicit go/no-go before building starts. If it's in, organizer-assigned teams (no draft/self-select) is the proposed MVP simplification. Confirm this is acceptable.

3. **Custom-per-person goal mode: build alongside head-to-head/collaborative, or sequence it after?**

4. ~~**Who's building?**~~ **Resolved:** Bryan is building (vibe coding). Andrew gates technical structure/architecture approval. Tyler owns testing. All three aligned on branding and MVP feature scope.

5. **Payments legality + Stripe legal review — do this before building the payments phase.** Skill-based contests are generally legal and unregulated in Canada, distinct from chance-based gambling. With Stripe in MVP scope, a real legal consult is warranted before launch — especially for org-sponsored pools at scale. Stripe's ToS also requires review for prize/escrow flows specifically. This should happen in parallel with early build phases so it doesn't block launch.

6. ~~**Zero-completers pot: refund or alternative?**~~ **Resolved:** full pro-rata refund — voids the challenge's stake transactions entirely rather than routing to an organizer-chosen alternative.

7. **Challenge close trigger: auto or manual?** The PRD assumes the challenge auto-closes on `end_date` and computes the settlement ledger automatically. Is there value in the organizer having a manual "close and settle" button — e.g., if the group wants to end early or extend by a day? Or is auto-close on end_date always the right call?

8. **Onboarding badge without Strava: alternative path or just skip it?** Users who don't connect Strava can't earn the onboarding badge as defined. Options: (a) skip it — they can earn future badges, (b) offer an alternative first-badge criteria (e.g., submit your first proof of any type), or (c) make the badge non-Strava-specific and change criteria to "complete any proof in your first week." Option (c) keeps the badge meaningful without requiring Strava.

9. **Timezone default at challenge creation.** Defaults to organizer's browser timezone. Confirm this works for the group — any members who travel internationally for an extended period during a challenge would be on a different clock.

10. **Build order sign-off.** See §9 for the proposed phase breakdown — confirm this sequence works for the team before building starts.

11. **Branding/design approval timing.** See §6. Does the branding/design pass (discuss → draft directions in Claude Design → team approval) complete before Phase 1 build starts, or run in parallel with early Phase 1 work? Recommend locking the direction before UI-heavy Phase 1 work begins to avoid rebuilding screens against a changed direction.

---

## 9. Build Sequence (proposed)

Build in vertical slices — each phase should be demo-able end to end before the next starts. Later phases gate on Stripe legal review completing in parallel.

**Phase 1 — Core loop (no money, no Strava)**
*Gated on: initial branding/design direction approved (see §6) — Phase 1 UI work builds against the approved design system.*

Auth (email + Google) → user profile → challenge creation (head-to-head mode only) → join via link + join code → proof submission (photo/text) → peer verification (auto-unless-challenged, 48h timeout) → leaderboard → in-challenge feed with reactions

*Exit criteria: a real challenge can be created, joined by multiple people, proofs submitted and verified, leaderboard updates correctly.*

**Phase 2 — Discovery + social layer**
Discovery tab with lifecycle filtering → public/private visibility → profile pages (badges, history, active challenges) → onboarding badge flow → Strava OAuth + webhook + backfill → 1-on-1 duel challenges

*Exit criteria: new users can find and join challenges; Strava proof flows end to end; duels work.*

**Phase 3 — Stakes + settlement** *(gate on legal review completing)*
Stripe Checkout stake collection at join → settlement ledger computation → payouts at close → zero-completers refund → manual refund override → organizer-configurable payout rules

*Exit criteria: a staked challenge can be run start to finish with real Stripe test payments and correct payouts.*

**Phase 4 — Additional modes**
Collaborative mode → custom-per-person mode → email notifications (dispute raised, challenge closing, results)

*Exit criteria: all three non-team modes work correctly with the verification and leaderboard engine.*

**Phase 5 — Stretch (cut here if timeline is tight)**
Team-based challenges → co-admin whitelist

---
