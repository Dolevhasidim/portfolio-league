# Production readiness plan — ליגת התיקים (Part 3)

Scenario from the assignment: tomorrow this goes live, you're the only ones
responsible — thousands of players, spikes at market open and after pushes,
developers shipping daily, and you're on holiday next week.

## 1. Scaling

- **Price feed:** fetch once per market per interval, cache centrally (don't let
  every client hit the market-data source). US + TA prices and the USD–ILS rate
  are read-heavy, write-rare — cache with a short TTL and serve from the edge.
- **Leaderboard:** never compute percentiles on read. Precompute the weekly
  board on a schedule (e.g. every few minutes + on market close) and store the
  result; reads are a single document fetch. Return and max-drawdown percentiles
  are computed over the same snapshot so the 0.85 / 0.15 score is consistent.
- **Spikes:** market open and push-driven logins are the two peaks. Pre-warm the
  cache before open, keep the morning briefing cheap (one scheduled job that
  fans out, not per-user computation), and rate-limit order placement per user.
- **Firebase / Vercel free tiers:** stay inside them by design — no per-client
  polling (use the cached snapshot), cap Firestore reads per session, and set
  billing alerts at 50% / 80% of the free quota so a spike can't silently
  become a bill.

## 2. Data integrity — the money rules

- **Idempotent orders:** every order carries a client-generated idempotency key;
  the server applies it once, even if the client retries. No double-execution.
- **No cancellation, no short, no leverage, 20% cap:** enforce all of these
  server-side (Firestore rules + Cloud Function), never trust the client —
  the client check is UX only.
- **Execution timing:** an order placed in trading hours fills at that day's
  close; outside hours, at the next trading day's open. Store the intended
  execution slot on the order at placement time so a delayed worker can't
  re-price it at the wrong close.
- **Audit log:** append-only record of every order, fill, price snapshot and
  score computation, so any dispute ("my order filled at the wrong price") is
  answerable from the log, not from memory.
- **Drawdown:** track each portfolio's peak; max drawdown is derived from the
  same price snapshots as returns, so return and stability percentiles can't
  disagree about what happened.

## 3. Observability (while on holiday)

- **Alerts, not dashboards:** alert on — order-fill error rate, price-feed
  staleness (> X minutes without a fresh snapshot), leaderboard job failures,
  auth failures, and free-tier usage crossing 80%.
- **Error budget:** define one (e.g. 99.5% of order placements succeed within
  2s over a rolling week). Burn-rate alerts page before the budget is gone.
- **Runbook:** one page per alert — symptom, how to confirm, the safe action
  (usually: pause the job / roll back / freeze trading), and who to call.
  The person on holiday should only be called for the "freeze trading"
  class of incident.
- **Logging:** structured logs with user id, order id and execution slot on
  every line, so a single order can be traced end to end.

## 4. Deploys — developers shipping daily

- **Preview environments:** every PR gets a Vercel preview; the world clock
  and prices in previews use a fixed seed so a reviewer sees a deterministic
  game.
- **Feature flags:** the morning briefing, new securities and scoring tweaks
  ship behind flags, so a bad release is a flag flip, not a redeploy.
- **Rollback:** one-command rollback to the previous Vercel deployment +
  Firestore rules version; the game stays playable on the old client during
  a rollback (API stays backward compatible for at least one version).
- **Migrations:** any change to the order or portfolio schema ships with a
  backfill job and is tested against a copy of production data first.

## 5. Security

- **Auth:** Firebase Auth only; no custom password storage. Every write is
  authorised by Firestore security rules keyed to the authenticated user.
- **Repo access:** the repo is public but `rotemyavne7`'s write access is
  verified, not assumed; no secrets in the repo — all keys live in Vercel /
  Firebase environment config.
- **Client trust:** the client never decides prices, fills or scores; it only
  displays what the server computed.

## 6. Cost guardrails

- Free-tier caps on Firebase (reads/writes) and Vercel (bandwidth, function
  invocations) with alerts at 50% and 80%.
- The most expensive operations — price fetching and leaderboard computation —
  run on a schedule, not per user, so cost grows with time, not with players.
- Kill-switch: a flag that degrades gracefully (e.g. briefing off, leaderboard
  refreshes hourly instead of every few minutes) if usage approaches a cap.
