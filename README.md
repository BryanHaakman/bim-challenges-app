# BIM Challenges

A fitness challenge platform built for the Bois in Motion community.

Born from the December challenge — streaks, dares, workouts, stakes — that ran through a group chat and a spreadsheet. Engagement was way higher than expected. The spreadsheet broke. This app fixes that, without killing the scrappy, social vibe that made it work.

---

## What We're Building

A mobile-first web app where any BIM member can create a fitness challenge, invite the group, submit proof, verify each other's work, and settle up. No spreadsheets. No chasing people for e-transfers in the group chat.

**Four challenge modes:**
- **Head-to-head** — everyone chases the same goal, ranked against each other
- **Collaborative** — group works toward a shared target, contribution breakdown per person
- **Custom-per-person** — everyone sets their own goal (what ran in December), scored as % progress
- **Team** — groups compete against groups (stretch goal for MVP)

**Proof & verification:** photo, video, text, or auto-pulled from Strava. Peer verification with configurable modes — disputes pause a submission until an organizer resolves it. Undisputed submissions auto-approve after 48 hours.

**Stakes & settlement:** optional buy-in collected via Stripe Checkout at join time. On close, the app computes the settlement ledger and triggers automatic payouts to winners via Stripe Connect. No manual e-transfers — it's handled in-app.

**Social layer:** per-challenge feed with emoji reactions, user profiles, badges, leaderboards, and 1-on-1 duels.

---

## Stack

- **Next.js** (App Router) — frontend + server actions
- **Supabase** — Postgres, Auth (email + Google), Storage
- **Vercel** — hosting, deploy target: `[name].boisinmotion.com`
- **Strava** — OAuth + webhook (reusing existing BIM client ID)
- **Stripe** — Checkout for stake collection, Connect for winner payouts

---

## Build Sequence

| Phase | What ships | Gate |
|---|---|---|
| **1 — Core loop** | Auth, challenge creation (head-to-head), join, proof submission, verification, leaderboard, feed | — |
| **2 — Discovery + social** | Discovery tab, profiles, onboarding badge, Strava integration, 1-on-1 duels | — |
| **3 — Stakes** | Stripe Checkout, Connect onboarding, ledger, auto-payouts | Legal review complete |
| **4 — More modes** | Collaborative, custom-per-person, email notifications | — |
| **5 — Stretch** | Team challenges, co-admin, majority vote verification | Timeline permitting |

---

## Docs

- [`PRD.md`](./PRD.md) — full product requirements, data model, open questions, and build sequence

---

## Open Questions (settle before building)

1. **App name** — decide before the first commit. Shortlist: Challange, App, Grind, Pact, Grit, Stakes, Ante, Reps, Commit.
2. **Team mode** — MVP or first cut if timeline is tight?
3. **Custom-per-person mode** — build in parallel with head-to-head/collaborative, or sequence after?
4. **Who's building?** — dev help vs. feature input only, to set a real timeline.
5. **Payments legal review** — needs to happen before Phase 3 starts, ideally in parallel with Phase 1–2.
6. **Zero-completers pot** — refund everyone (proposed default) or organizer picks an alternative?
7. **Challenge close** — auto on end date, or manual organizer trigger?
8. **Onboarding badge without Strava** — skip it, or alternative criteria for non-Strava users?

See [`PRD.md §7`](./PRD.md) for full discussion on each.
