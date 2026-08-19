# BIM Challenges — Product Requirements Document (MVP)

**Status:** Draft for team review
**Target:** [name].boisinmotion.com (subdomain, Vercel)

---

## 1. Problem & Opportunity

The December challenge (streaks, dares, workouts, self-organized in group chat + spreadsheet) proved the concept — engagement was way higher than expected. Manual tracking broke down at two points: no single source of truth for verification, and money/stakes were a headache to calculate and collect.

The app replaces the spreadsheet + group chat with structure, without killing the scrappy, social vibe that made December work — the feed and social layer are core, not decoration.

**Explicitly single-tenant for now.** No multi-group support, no friends/follow system, no DMs. This is BIM's app for BIM's circle. Multi-group is a future direction but not architected in yet beyond keeping the schema sane.

**Long-term directions (not MVP, but shape the data model):**
- Group training tied to a specific race (e.g. a group training together for a marathon) — would need training-plan adherence tracking (weekly mileage targets, etc.), parked for later
- AI/computer-vision proof verification (e.g. counting reps from video)

---

## 2. MVP Scope

### Core user stories (five original + additions from working session)
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

### Explicitly out of scope for MVP
- Multi-group / multi-tenant support
- Native mobile app (mobile-responsive web only)
- Push notifications (email only)
- Friends/follow system, direct messaging
- Badge rules/catalog beyond the one onboarding badge (schema should support more, logic is phase 2)
- AI/computer-vision proof verification
- Training-plan adherence tracking for race-training groups

---

## 3. Users & Roles

| Role | Description |
|---|---|
| **Organizer** | Any user can create and manage a challenge through to completion. No special account tier required. |
| **Co-admin** | Organizer can whitelist other participants as co-admins to help manage the challenge and resolve disputes |
| **Participant** | Joins a challenge, submits proof, verifies/disputes others' proof |

No public/guest browsing beyond discovery previews — signup is required to join a challenge, not required to preview a challenge via invite link.

---

## 4. Core User Stories (detailed)

### 4.1 Create & Join a Challenge
- Organizer sets: name, description, **mode** (head-to-head / collaborative / custom-per-person / team), goal type (streak, distance, time, dare), start/end date, proof rules, verification mode, stakes (optional), payout rule (if staked), visibility (public/private), team structure (if team mode)
- Any user can create a challenge — no gating
- Join via: direct invite link (primary acquisition path — preview challenge details before any signup prompt), join code, or discovery tab
- New-user signup is deferred until they actually initiate joining, not shown upfront

### 4.2 Challenge Modes
Three shapes, in priority order for MVP:

- **Head-to-head**: everyone shares the identical goal, ranked against each other. Ties are allowed and left as ties, no tiebreaker.
- **Collaborative**: group works toward one shared target. Leaderboard shows both the overall shared progress bar AND a per-person contribution breakdown underneath.
- **Custom-per-person**: each participant sets their own goal (e.g. one person runs 5k/day, another attempts an Ironman-distance total, another goes for 100k in a day) — same mechanic that ran in December. More complex to build (no common unit for ranking), so scored as % progress toward each person's own target.
- **Team**: multiple people grouped into a team, competing against other teams (e.g. 2v2). Team score aggregates member proof.

### 4.3 Submit Proof
- Photo, video, text note, or auto-pulled Strava activity (OAuth + webhook)
- Strava supports **retroactive backfill** — activities from before someone connected Strava can still count if they fall within the challenge window
- Strava-sourced proof auto-verifies (skips peer verification) unless flagged

### 4.4 Peer Verify / Dispute
- Verification mode is an **organizer-configurable setting per challenge**: single approval or auto-approve-unless-challenged (default recommendation — lowest friction, only creates work on actual disputes)
- A disputed submission is **paused** — does not count toward the leaderboard until resolved. Resolved items are given the submission timestamp. If a challenge could be over but a pending item is blocking challenge completion, it will be marked as pending.
- Resolution authority: organizer by default, or any co-admin they've whitelisted for that challenge

### 4.5 Streak / Leaderboard
- Server-computed and cached per challenge
- Head-to-head: ranked by raw progress toward the goal
- Collaborative: group total + individual contribution breakdown
- Custom: % progress toward each person's own target
- Team: aggregated team score, ranked against other teams

### 4.6 Settle Up
- **Stripe handles stake collection and payout** — participants pay their stake via Stripe Checkout at join time; funds are held and disbursed at challenge close via Stripe Connect
- **Payout rule is organizer-configurable per challenge** (winner-take-all, split among all who completed their goal, or other) — not a single fixed app-wide rule
- On challenge close, the server computes the settlement ledger and triggers Stripe payouts to winners automatically
- **No refunds** if a participant drops out or goes quiet — stake stays in the pot by default
- **Exception**: organizer can manually trigger a Stripe refund for legitimate cases (injury, group-agreed fairness) — separate override from the default no-refund rule
- Two stake models: peer-funded (participants pay into their own pot via Stripe) and org-sponsored (BIM or sponsor funds a prize pool) — Stripe Connect handles disbursement for both. Prizes could also be physical goods (gear, nutrition, race entries) with manual fulfillment.

### 4.7 Discovery
Discovery tab reflects challenge lifecycle status, filterable by:
- **Upcoming** — not started yet, joinable or request-to-join
- **Active, open** — in progress, still accepting new joiners
- **Active, closed** — in progress, not accepting new joiners (spectate-only view for non-participants)
- **Closed/archived** — disappears from discovery, still viewable from a participant's profile/history

Also supports creating a new challenge directly from this tab.

### 4.8 Profile
Includes: profile picture, name, badges, history of completed challenges, current active challenges. Clickable from anywhere a name/avatar appears (leaderboard, feed) — opens the profile and offers a "challenge this person" action, feeding into the 1-on-1 duel flow.

### 4.9 1-on-1 Duels
Direct challenge between two users, using the same proof/verification/leaderboard system as group challenges, just scoped to two participants. Discoverable from any profile view.

### 4.10 Onboarding
First-run flow: connect Strava, complete one logged activity within the first week → awards first badge. Functions as a built-in tutorial challenge to prevent signup-then-nothing drop-off.

---

## 5. Feature Set (MVP)

| Feature | Priority | Notes |
|---|---|---|
| Auth (email or Google) | P0 | Supabase Auth |
| User profiles (photo, name, badges, history) | P0 | |
| Challenge creation (all 4 modes) | P0 | Head-to-head + collaborative first; custom-per-person and team can slip if timeline is tight |
| Join via link/code/discovery | P0 | |
| Discovery tab with lifecycle filtering | P0 | |
| Proof submission (photo/video/text) | P0 | Supabase Storage |
| Strava OAuth + webhook + retroactive backfill | P0 | Reuse existing BIM Client ID |
| Peer verification/dispute (3 configurable modes) | P0 | |
| Streak/leaderboard computation (per mode) | P0 | Server-authoritative |
| In-challenge social feed | P0 | Visible to all participants in that challenge |
| Stripe Checkout — stake collection at join time | P0 | Stripe Connect for disbursement |
| Ledger/settlement calculation w/ configurable payout rule | P0 | Server-triggered Stripe payout on close |
| Withdrawal + no-refund default + manual Stripe refund override | P0 | |
| 1-on-1 duel challenges | P0 | Same engine, 2-participant scope |
| Onboarding challenge + first badge | P0 | |
| Team-based challenges (2v2 etc.) | P1 | Stretch goal — try for MVP, flag as first thing to cut if timeline slips |
| Custom-per-person goal mode | P1 | Can follow head-to-head/collaborative if needed |
| Email notifications (dispute raised, challenge closing, results) | P1 | No push for MVP |
| Co-admin whitelist per challenge | P1 | |
| Badge catalog beyond onboarding badge | P2 | Schema-ready, rules TBD |
| AI/computer-vision proof verification | P2 | Phase 2/3 — design proof storage (esp. video) to not preclude this later |
| Race-training group mode (adherence tracking) | P2 | Explicitly parked, noted for future |
| Org-sponsored prize pools | P2 | Ledger model supports it; real build TBD |

---

## 6. Technical Architecture

**Stack:** Next.js (App Router) + Supabase (Postgres, Auth, Storage) + Vercel — matches existing haakman.ca stack.

```
[name].boisinmotion.com (Vercel)
  ├─ Next.js frontend + server actions
  ├─ Supabase Auth (email + Google OAuth)
  ├─ Supabase Postgres (challenges, proofs, verifications, ledger, teams)
  ├─ Supabase Storage (proof photos/video)
  ├─ Strava OAuth + webhook subscription
  └─ Stripe (Checkout for stake collection, Connect for payouts, webhooks for payment events)
```

### Data model (MVP, updated)

```
users
  id, email, display_name, avatar_url, strava_athlete_id, strava_tokens (encrypted),
  stripe_account_id (Connect account for receiving payouts)

challenges
  id, organizer_id, name, description,
  mode (head_to_head | collaborative | custom | team),
  goal_type (streak|distance|time|dare),
  start_date, end_date, proof_rules (jsonb),
  verification_mode (single|majority|auto_unless_challenged),
  stake_amount, payout_rule (jsonb),
  visibility (public|private), status (upcoming|active_open|active_closed|closed),
  join_code

challenge_admins
  id, challenge_id, user_id   -- co-admin whitelist

challenge_participants
  id, challenge_id, user_id, team_id (nullable), joined_at,
  status (active|withdrawn|withdrawn_refunded), personal_goal (jsonb, for custom mode)

teams
  id, challenge_id, name

proofs
  id, challenge_id, participant_id, submitted_at,
  type (photo|video|text|strava), content_url, strava_activity_id,
  status (pending|verified|disputed|rejected)

verifications
  id, proof_id, verifier_id, vote (approve|dispute), reason, created_at

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
- **Strava via webhook subscription**, not polling — matches activity to challenge rules server-side before auto-verifying, must support backfilled/past-dated activities within the challenge window
- **Disputed proof pauses standings recompute** for that participant until resolved
- **Team mode aggregates member-level standings into a team-level rank** — build the individual layer first, team is a rollup on top
- Proof storage should assume video may later feed a CV pipeline — don't discard resolution/format in a way that would block that later
- **Stripe Checkout** collects stakes at join time; **Stripe Connect** handles payout disbursement to winners — Stripe webhooks confirm payment success before participant is marked active in a staked challenge
- Stripe account onboarding (Connect) required for any user who may receive a payout — prompt at challenge-close if not yet connected

---

## 7. Open Questions for the Team (Andrew, Tyler, Tomek)

1. **Name.** Shortlist: App, Grind, Pact, Grit, Stakes, Ante, Reps, Commit.
2. **Team mode: MVP or cut first if timeline is tight?** It's flagged P1/stretch — worth a explicit go/no-go before building starts.
3. **Custom-per-person goal mode: build alongside head-to-head/collaborative, or sequence it after?**
4. **Who's building?** Confirm Tomek/Andrew/Tyler's role — dev help vs. feature input only — to set a real timeline.
5. **Payments legality** — skill-based contests are generally legal and unregulated in Canada, distinct from chance-based gambling. With Stripe now in MVP scope, worth a real legal consult before launch — especially for org-sponsored pools at scale. Stripe's ToS also requires review for prize/escrow flows.

---

## 8. Build Prompt (for Claude Code / scaffolding)

> Build a Next.js (App Router) + Supabase app called [NAME] for a friend-group fitness challenge platform. Any user can create a challenge in one of four modes: head-to-head (shared goal, ranked against each other, ties allowed), collaborative (shared group target with an overall progress bar plus per-person contribution breakdown), custom-per-person (each participant sets their own goal, scored as % progress), or team (teams aggregate member scores and compete against other teams). Participants join via invite link (preview-before-signup), join code, or a discovery tab filtered by lifecycle status (upcoming / active-open / active-closed-spectate-only / archived). Proof submission via photo/video/text upload (Supabase Storage) or auto-pulled Strava activity (OAuth + webhook, with retroactive backfill support for activities before the user connected Strava). Verification mode is organizer-configurable per challenge: single approval, majority vote, or auto-approve-unless-challenged (disputed proof pauses out of standings until resolved by the organizer or a whitelisted co-admin). Server computes streaks, completion %, and per-mode leaderboards — never trust client-submitted values. Optional stakes per challenge with an organizer-configurable payout rule (winner-take-all, split among completers, etc.); stakes are collected via Stripe Checkout at join time and held until challenge close, at which point the server computes the settlement ledger and triggers Stripe Connect payouts to winners automatically. No refunds on dropout by default; organizer can manually trigger a Stripe refund as an override. Include a first-run onboarding challenge (connect Strava + log one activity in week one → first badge) and a full in-challenge social feed visible to all participants. Include user profiles (photo, name, badges, history, active challenges) clickable from anywhere a user appears, supporting a direct 1-on-1 "duel" challenge action. Auth via Supabase (email + Google). Mobile-first responsive UI, three-tab structure: Your Challenges / Discover / Profile. Deploy target: Vercel, subdomain [name].boisinmotion.com.

---
