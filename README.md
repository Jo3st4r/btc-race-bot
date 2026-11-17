# BTC race bot — btc-race-24h-20261117-0603

Dip-buy bot for the make.fun BTC race (bars-replay/v1, cbBTC/USD 1-minute
bars from Orca Whirlpool, $1,000 paper capital).

## Strategy

Long only, one symbol (CBBTC), integer math only, fully deterministic.

- **Entry:** when the close is **1.9% or more below the highest close of
  the last 45 minutes**, buy with the full cash.
- **Exit:** sell the whole position when its value is **1.5% or more
  above its cost**. No trailing stop.
- **Fallback:** if no position was ever held by bar 1200 (a day so quiet
  or so steadily rising that no 1.9% dip appeared), buy with the full cash
  and hold to the flatten, so the entry always clears the 1-trade gate.

## Why these numbers

The dev day is a quiet range day (start $64,461, end $64,776, max close
$65,467, min $63,874). A round trip costs ~37 bps (4 bps fee + 2 bps
slippage + ~12 bps impact per side at ~790k USDC depth), so a trade needs
a move well above that; frequent trading only bleeds fees (the kit's
ema-cross baseline loses $106.21 on this window in 68 trades, while
buy-and-hold makes $1.07 and momentum $2.45).

The parameters were chosen in the middle of the plateau that is
profitable on **both** frozen 24h windows available for testing — this
race's dev window and the 15 Nov race window (a crash + V-recovery day):

| parameter | too shallow / short | chosen | too deep / long |
|---|---|---|---|
| dip vs 45-min high | 1.5% buys the falling knife on the crash day (−$15.11) | **1.9%** | 2.1% never fires on the quiet day (0 trades) |
| take-profit vs cost | 1.0% exits too early (+$8.50) | **1.5%** | 1.7% never fires on the crash day |
| lookback | 30 min: no dip on the quiet day | **45 min** | 75+ min: buys too early into the crash |

A trailing stop (even a wide 3% one) was tested and rejected: on both
windows it only shook the bot out of the recovery and turned winners
into churn.

## Dev grade (official harness)

Graded with the kit's own harness engine (`sim/bars/engine.ts`) driving
`main.py` as a subprocess — the kit's `grade-local --driver process` is
node-only, so the harness was driven directly. The driver was validated
first against the official grades: buy-and-hold reproduces exactly
(+$1.067187, 1 trade, $0.800588 fees, max DD 0.021189434).

| window | net PnL | trades | fees | max drawdown |
|---|---|---|---|---|
| dev (17 Nov) | **+$14.004681** | 2 | $0.805765 | 0.62% |
| 15 Nov (cross-check) | **+$13.633628** | 2 | $0.805616 | 4.37% |

Dev gate (net PnL > $0): **PASS** on both windows. Two runs on the dev
window give byte-identical results (deterministic). No rejected orders,
no abort, no future bars.

## Files

- `main.py` — the bot (Python 3.13, standard library only).
- `vendor/makefun_bars.py` — byte-identical copy of the kit's
  `sdk/makefun_bars.py` (the bars-replay/v1 Python shim).
- `LICENSE` — MIT.
