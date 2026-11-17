[![make.fun · btc-race-24h-20261117-0603](https://make.fun/badge/btc-race-24h-20261117-0603.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)
[![rank](https://make.fun/badge/btc-race-24h-20261117-0603/Indodegen/cbbtc-race-bot/rank.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)
[![reward](https://make.fun/badge/btc-race-24h-20261117-0603/Indodegen/cbbtc-race-bot/reward.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)

Built for the [make.fun bounty](https://make.fun/bounty/btc-race-24h-20261117-0603) — Race a BTC trading bot live for 24 hours.

# Swing Ladder — cbBTC 24h race bot

A long-only swing bot for Coinbase Wrapped BTC (cbBTC/USDC) on the
make.fun 24-hour race. It buys $1,000 of CBBTC once 13 closes have been
seen, then harvests the day's repeated up-legs: sell the whole position
when the close is 90 bps above the entry close, rebuy in full when the
close has pulled back 110 bps below the last exit close, and cap every
leg with a 150 bps hard stop below the entry close and a 200 bps
trailing stop below the highest close since entry. After 1,321 closes
it locks in any remaining gain of 30 bps or more instead of gambling
the last two hours.

## Why this shape

The dev window is a rising channel (about $64.0k–$65.5k) whose up-legs
run roughly 80–100 bps between pullbacks of 100–170 bps, while a
full-size round trip costs about 26–38 bps in fees, slippage and price
impact. A 90 bps take-profit clears the round-trip cost with room to
spare; a 110 bps dip filter only re-enters after a real pullback, not
noise; the stops bound the loss on a day that trends down instead.

The same parameters stayed positive across the dev window, its time
reversal, +75/−75 and +150/−150 bps drifted copies of it, and
block-bootstrapped reshuffles of it — so the edge is the channel
structure, not one lucky leg.

## Dev-window result (exact local replay of the kit's fill model)

- Net PnL: **+$26.12** (26,116,787 micro-USDC)
- Trades: 8 (4 buys, 4 sells), fees $3.21, max drawdown 0.65%
- vs baselines on the same window: buy-and-hold +$1.07, momentum
  +$2.45, ema-cross −$106.21, rsi-bollinger −$24.25

## Implementation

`main.py` speaks `agent-fun/bars-replay/v1` directly (NDJSON on
stdin/stdout, no SDK shim): one reply per batch, exact integer
arithmetic on closes only — no floats, no wall clock, no environment,
no file reads, no randomness. Two runs on the same data therefore
produce byte-identical action logs. It needs no history: entry close,
peak close, last exit close and a bar counter are the whole state.
