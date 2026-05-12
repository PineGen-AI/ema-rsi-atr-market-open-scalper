# Market Open Scalping Strategy (EMA + RSI + ATR)

## Overview

This strategy demonstrates an intraday scalping framework focused on the market open. It limits trading to a defined session window and uses a combination of EMA structure, RSI filtering, and ATR-based risk parameters. The goal is to provide a clear, rule-based template suitable for backtesting and experimentation.

## What it does

The strategy evaluates short-term directional conditions using:

Fast/slow EMA structure

RSI as a momentum filter

ATR for stop placement and target sizing

Session/time filtering to restrict execution to the opening window

It is intended for intraday use (1m–5m charts) and can be adjusted per market.

## Core logic

### 1) Session filter (market open window)

Trades are allowed only during a user-defined session time (default example: 09:30–11:00, exchange time).

This concentrates signals into a consistent intraday period.

### 2) Trend confirmation (EMA structure)

Long bias when fast EMA crosses above slow EMA

Short bias when fast EMA crosses below slow EMA

### 3) Momentum filter (RSI)

RSI is used as a directional/momentum condition rather than a standalone overbought/oversold signal.

Users can adjust RSI thresholds to fit different symbols and regimes.

## Entries & risk management

Entries are evaluated on confirmed bars

One position at a time (if configured)

Stop-loss uses an ATR multiple

Take-profit is defined using a fixed risk–reward ratio (or configurable target logic)

## Usage & customization

Users can experiment with:

Session window length and times

EMA lengths (fast/slow)

RSI thresholds

ATR length and multipliers

Risk–reward settings

## Notes

This script is provided for educational/testing purposes. Results vary by symbol, timeframe, and market conditions. Always validate across multiple instruments and forward-test before using alerts or live execution.

## Automate It with PineGen AI

Take this strategy to the next level with PineGen AI automation:

Website: https://www.pinegen.ai/

Twitter: https://x.com/PineGenAI

Telegram Channel: https://t.me/PineGenAI

YouTube Channel: https://www.youtube.com/@pinegenai

## Pine Script Code

```pine
//@version=6
strategy(
    "High-Probability Scalper (Market Open)",
    overlay=true,
    calc_on_every_tick=true,
    initial_capital=10000
)


// ───── Inputs
fastEmaLen = input.int(9, "Fast EMA", minval=1)
slowEmaLen = input.int(21, "Slow EMA", minval=1)
rsiLen     = input.int(14, "RSI Length", minval=1)
atrLen     = input.int(14, "ATR Length", minval=1)
riskRR     = input.float(1.5, "Risk / Reward", step=0.1)


// ───── Indicators
emaFast = ta.ema(close, fastEmaLen)
emaSlow = ta.ema(close, slowEmaLen)
rsiVal  = ta.rsi(close, rsiLen)
atrVal  = ta.atr(atrLen)


// ───── Market Open Session (NYSE first 90 minutes)
inSession = not na(time(timeframe.period, "0930-1100"))


// ───── CROSS CONDITIONS (GLOBAL — FIXES WARNING)
bullCross = ta.crossover(emaFast, emaSlow)
bearCross = ta.crossunder(emaFast, emaSlow)


// ───── Entry Conditions (SAFE)
longCondition  = inSession and bullCross and rsiVal > 40 and rsiVal < 70 and barstate.isconfirmed
shortCondition = inSession and bearCross and rsiVal < 60 and rsiVal > 30 and barstate.isconfirmed


// ───── Entries
if longCondition
    strategy.entry("Long", strategy.long)


if shortCondition
    strategy.entry("Short", strategy.short)


// ───── Risk Management (ATR based)
longStop  = close - atrVal
longLimit = close + atrVal * riskRR


shortStop  = close + atrVal
shortLimit = close - atrVal * riskRR


strategy.exit("Long Exit",  from_entry="Long",  stop=longStop,  limit=longLimit)
strategy.exit("Short Exit", from_entry="Short", stop=shortStop, limit=shortLimit)


// ───── Plots
plot(emaFast, title="Fast EMA", color=color.green)
plot(emaSlow, title="Slow EMA", color=color.red)
```
