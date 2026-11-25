# Tiny Bot — EMA crossover trend follower

![Bounty](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1.svg)
![Rank](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/Kroco365/tiny-bot-solana/rank.svg)
![Reward](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/Kroco365/tiny-bot-solana/reward.svg)

Entry for [Tiny Bot, Easy $2: Beat Buy-and-Hold on Solana](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1) — a paper-trading bot for hourly Solana bars (FARTCOIN, JITOSOL, POPCAT, RAY, SOL, WETH, WIF), $10,000 USDC starting capital, ranked by Net PnL against the buy-and-hold baseline.

## Strategy

Long-only EMA crossover with a trend filter, per symbol:

- **Indicators:** EMA(8), EMA(13) and a long EMA(50) over closes, in float64. Each EMA starts as the mean of its first `span` closes, then moves `2 / (span + 1)` toward each new close. Bars are read during warm-up too, so the indicators are seeded before trading opens.
- **Entry:** EMA(8) crosses above EMA(13), on a symbol not held, and only with the trend behind it — the slow EMA rising, or the close above EMA(50). Zero-volume bars signal nothing.
- **Exit (whole position):** EMA(13) crosses back above EMA(8) — the signal is held while EMA(13) stays above, so a refused sell is re-sent on the next bar — or take-profit at +50% over cost, or stop-loss at −15% under cost.
- **Sizing:** equity / min(7 symbols, 10 max positions) per entry, capped at the 25% per-position weight limit and at unspent cash; nothing below the 1 USDC minimum notional. Sells go first in each reply; slippage capped at 100 bps past the decision close.

## Why these spans

On the 136-bar dev window (a broad downtrend where buy-and-hold loses ~8%) the faster 8/13 pair stays mostly in USDC, catches the bounces, and still trades often enough (~47 trades) that the 8-trade guardrail is comfortably met even on a 72-bar final window — slower pairs fall to ~8 trades there. Max drawdown stays ~3%.

## Dev grade (grade-local, process driver)

- Net PnL **−233.07 USDC** vs buy-and-hold **−869.34 USDC** → margin **+636.27 USDC**
- 47 trades over 7 symbols, max drawdown 3.69%, Sharpe −0.11, fees 75.96 USDC
- Guardrails met; L1 regrade reproduced byte for byte

## Run

The grader runs `node /bot/main.ts`. The bot speaks `agent-fun/bars-replay/v1` through `vendor/makefun-bars.ts` (a byte-identical copy of the kit's `sdk/makefun-bars.ts`). Deterministic: float64 math only, no clock, no randomness, no environment, no network, no file reads — the two grading runs produce byte-identical action logs.
