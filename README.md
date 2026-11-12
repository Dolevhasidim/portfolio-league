# ליגת תיקים — Portfolio League

A playable virtual-portfolio league game. Each player starts with **₪1,000,000**,
buys and sells TA + US securities, and competes on a **weekly leaderboard (Mon–Fri)**.

**Live link:** https://dolevhasidim.github.io/portfolio-league/
**Repo:** https://github.com/Dolevhasidim/portfolio-league — public GitHub repo with Write access for `rotemyavne7` (invite sent, pending acceptance).

## How to play

1. Open the live link — no install, no sign-up needed for the demo.
2. Place buy/sell orders in shekels (fractions allowed). Orders execute at that
   day's close if placed during trading hours, or at the next trading day's open
   if placed outside hours. Orders can't be cancelled.
3. Advance the world clock to the next day to see orders fill and prices move.
4. Check the morning briefing and your place on the weekly leaderboard.

## Rules

| Rule | Value |
|---|---|
| Starting capital | ₪1,000,000 |
| World | 20 securities (TA + US), prices over time, USD–ILS rate, advancable world clock |
| Buy / sell | In shekels, fractions allowed |
| Order in trading hours | Executes at that day's close |
| Order outside trading hours | Executes at next trading day's open, no cancellation |
| Trading hours (per market) | TA Sun–Thu 10:00–17:30, Fri 10:00–13:50 (Israel time); US 09:30–16:00 (New York time) — verified from the PDF, 2026-11-12 |
| US securities | Bought in dollars, converted at the USD–ILS rate at the moment of execution |
| Short / leverage | Not allowed |
| Max single position | 20% of portfolio at purchase |
| Leaderboard | Weekly, Mon–Fri |
| Score | 0.85 × return percentile + 0.15 × stability percentile (smaller max drawdown = better) |

⚠️ **Rules re-checked against the assignment PDF on 2026-11-12** (`assignment-original.pdf`, Gmail message 1a19d4fe732eaf07, Rotem Yavne, 2026-11-02 19:30 IST). The table above now matches the PDF, including trading hours and USD conversion at execution. The PDF states the rules are deliberately incomplete — the decisions below fill the gaps.

## Rules I completed (rule / my decision / why)

| Rule gap in the PDF | My decision | Why |
|---|---|---|
| No price file supplied; world must be built | Prototype world: 20 fixed securities (10 TA + 10 US, listed below) with simulated prices over time and a USD–ILS rate starting at 3.65, advancing day by day | No free live price feed was wired in the demo window; a deterministic simulated world lets every player share one clock and lets the game be played without waiting for real markets, as the PDF allows |
| How the world clock advances / who may advance it | Any player advances the shared world clock one trading day at a time from the game UI ("advance day"); no special permission needed in the demo | The PDF requires a way to advance the clock and that all players share it; a single shared advance button is the simplest honest implementation for a demo, and the permission question is answered here rather than left open |
| Registration | Demo: no sign-up — open the link and play immediately | Keeps the live link playable with no install, as required; real registration (Firebase Auth) is part of the port to the submission stack |
| What "stability" measures | Max drawdown (MDD): largest % fall from a portfolio peak to a later trough in the week; smaller = better (percentile 1 = smallest drawdown) | This is the PDF's own definition, confirmed on re-read |
| Order execution in the prototype | Advancing the day is the close: queued orders fill at that day's close | The prototype collapses intraday hours into a day step; per-market hours above are the target behaviour for the ported version |

## What I gave up, and why

- **Full port to TypeScript / React Native (Expo) / Firebase / Vercel in one day:** not attempted. The prototype proves the rules, scoring and retention feature as a playable static demo; an unported Expo skeleton with no live link would be a worse submission. The port is the explicit next step.
- **Live market prices:** simulated prices instead, for the reason above.
- **Real registration / multi-player leaderboard:** simulated rivals in the demo; the weekly board logic (0.85 return + 0.15 stability) is real.

## How the world was built, and why

20 securities, 10 TA (Teva, Bank Hapoalim, Bank Leumi, Israel Discount Bank, Mizrahi-Tefahot, Nice, Elbit Systems, Tower Semiconductor, ICL, Energix) and 10 US (Apple, Microsoft, Nvidia, Alphabet, Amazon, Tesla, S&P 500 ETF, Nasdaq-100 ETF, Berkshire Hathaway, Johnson & Johnson), with simulated daily prices and a moving USD–ILS rate. Chosen to satisfy the PDF's TA+US mix with recognisable names, without depending on a paid feed — the whole assignment must be doable for free, with no credit card, per the PDF.

Securities in the demo world (suggestion — adjust to what the PDF implies and to
data you can source for free): TA — Teva, Bank Hapoalim, Bank Leumi, Israel
Discount Bank, Mizrahi-Tefahot, Nice, Elbit Systems, Tower Semiconductor, ICL,
Energix; US — Apple, Microsoft, Nvidia, Alphabet, Amazon, Tesla, S&P 500 ETF,
Nasdaq-100 ETF, Berkshire Hathaway, Johnson & Johnson.

## Daily retention feature — morning market briefing

Every morning the player gets a 30-second briefing: what moved overnight, their
leaderboard movement, and one suggested action — plus a visit streak. It rides
the daily trading rhythm and the weekly board. It needs one scheduled job and
one push notification. Measured by: daily return rate (DAU), D1/D7 retention,
and streak-length distribution.

## Production readiness — summary

Full plan in [`production-plan-draft.md`](production-plan-draft.md). In short:
scale the price feed and leaderboard computation (cache prices, precompute
boards), make orders idempotent with an audit log, ship with preview envs,
feature flags and a rollback path, alert on an error budget with an on-call
runbook, and keep everything inside free-tier caps.

## Status / submission checklist

- [x] Playable prototype (single-file demo): buy/sell, 20% limit, leaderboard,
      world clock, morning briefing + streak — `ligat-tikim-prototype.html`
- [ ] Port to the submission stack (TypeScript, React Native / Expo, Firebase,
      Vercel) — the prototype proves the rules and scoring, it is not the
      submission stack
- [ ] Public GitHub repo, `rotemyavne7` has write access (check, don't assume)
- [ ] Live link opens in a browser, no install, playable end-to-end
- [ ] Every rule re-checked against the PDF
- [ ] Submitted before Thu 2026-11-12, 23:59 (Thu 20:00 tennis cancelled, evening free — no 20:00 constraint)
