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

**Proof & verification:** photo, video, text, or auto-pulled from Strava. Peer verification with configurable modes — disputes pause a submission until an organizer resolves it.

**Stakes & settlement:** optional buy-in collected via Stripe Checkout at join time. On close, the app computes the settlement ledger and triggers automatic payouts to winners via Stripe Connect. No manual e-transfers — it's handled in-app.

**Social layer:** in-challenge feed, user profiles, badges, leaderboards, and 1-on-1 duels.

---

## Stack

- **Next.js** (App Router) — frontend + server actions
- **Supabase** — Postgres, Auth (email + Google), Storage
- **Vercel** — hosting, deploy target: `[name].boisinmotion.com`
- **Strava** — OAuth + webhook (reusing existing BIM client ID)
- **Stripe** — Checkout for stake collection, Connect for winner payouts

---

## Docs

- [`PRD.md`](./PRD.md) — full product requirements, data model, open questions, and build order

---

## Open Questions (settle before building)

