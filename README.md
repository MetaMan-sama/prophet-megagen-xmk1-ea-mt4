# Prophet MegaGen Xmk1 — MQL4 Expert Advisor

A MetaTrader 4 **Expert Advisor (EA)** that performs fully autonomous multi-pattern recognition-driven trading by detecting a comprehensive library of **candlestick and chart patterns** across 40+ pattern types, filtering signals through an EMA trend filter and optional volume confirmation, and executing `OrderSend()` buy and sell orders with ATR-derived dynamic stop loss, configurable risk/reward take profit, and optional trailing stop management — all driven by a new-bar-only `OnTick()` execution gate.

---

## Overview

Prophet MegaGen Xmk1 is a production-structured pattern recognition Expert Advisor that combines three independent signal validation layers before executing a trade: pattern detection, trend alignment via EMA, and volume confirmation. The EA enumerates a 40+ member `PATTERN_TYPE` enum spanning single-candle reversals, multi-candle formations, and structural chart patterns, classifying each as bullish or bearish through dedicated `IsBullishPattern()` and `IsBearishPattern()` switch dispatchers. Trade entries are gated by a new-bar timestamp check to prevent multiple signals on the same candle, and open positions are managed each tick through `ManageTrades()` for trailing stop updates. Indicator handles are cleanly initialized in `OnInit()` and released in `OnDeinit()` following MT4 best practices for resource management.

---

## File Type

> **This file is a MQL4 Expert Advisor (EA) source file** distributed as `.txt` for portability.
> Rename to `New_Prophet_MegaGen_Xmk1.mq4` and place in `MQL4/Experts/` — **not** `MQL4/Scripts/`.
> EAs use `OnInit()` / `OnDeinit()` / `OnTick()` lifecycle functions and run continuously on every price tick, unlike scripts which execute once.

---

## Features

- **40+ pattern enum** — `PATTERN_TYPE` covers bullish reversals (Hammer, Engulfing, Morning Star, Three White Soldiers, Cup with Handle, Ascending Triangle, Bull Flag, etc.), bearish reversals (Shooting Star, Dark Cloud Cover, Evening Star, Three Black Crows, Head & Shoulders, Bear Flag, etc.), and structural chart patterns (Double Top/Bottom, Symmetrical Triangle, Wedges, Rectangles, Pennants)
- **EMA trend filter** — `IsBullishTrend()` / `IsBearishTrend()` validate price position relative to the `EMAPeriod` EMA via `iMA()` handle; bypassed when `UseTrendFilter = false`
- **Volume confirmation filter** — pattern detection functions check `iVolume()` against comparative bars where volume context adds reliability; bypassed when `UseVolumeFilter = false`
- **ATR-based dynamic stop loss** — `iATR()` handle computes volatility-scaled SL distance; TP calculated as `SL × RiskRewardRatio`
- **Optional trailing stop** — `ManageTrades()` adjusts open position SL on each tick when `UseTrailingStop = true`
- **New-bar execution gate** — `static datetime lastBarTime` compared against `iTime(NULL, 0, 0)` prevents redundant pattern evaluation within the same candle
- **Magic number isolation** — `CountOrders()` filters `OrdersTotal()` by both `Symbol()` and `MagicNumber` to prevent interference with manual or other EA trades
- **Handle-based indicator management** — `iMA()` and `iATR()` handles initialized in `OnInit()` with `INVALID_HANDLE` validation; released via `IndicatorRelease()` in `OnDeinit()`
- **5-digit broker normalization** — `point` adjusted by `×10` when `Digits == 3 || Digits == 5` at initialization

---

## Pattern Library

### Bullish Reversal Candlestick Patterns
Hammer, Inverted Hammer, Dragonfly Doji, Bullish Spinning Top, Bullish Kicker, Bullish Engulfing, Bullish Harami, Piercing Line, Tweezer Bottom, Morning Star, Bullish Abandoned Baby, Three White Soldiers, Bullish Three Line Strike, Morning Doji Star, Three Inside Up, Three Outside Up, Bullish Cross, Bullish Meeting Line

### Bearish Reversal Candlestick Patterns
Hanging Man, Shooting Star, Bearish Spinning Top, Gravestone Doji, Bearish Kicker, Bearish Engulfing, Bearish Harami, Dark Cloud Cover, Tweezer Top, Evening Star, Bearish Abandoned Baby, Three Black Crows, Bearish Three Line Strike, Evening Doji Star, Three Inside Down, Three Outside Down, Bearish Harami Cross

### Chart / Structural Patterns
Double Bottom, Double Top, Head & Shoulders, Inverted Head & Shoulders, Ascending Triangle, Descending Triangle, Symmetrical Triangle, Bull Flag, Bear Flag, Bullish Pennant, Bearish Pennant, Cup with Handle, Inverted Cup, Bullish Rectangle, Bearish Rectangle, Bullish Wedge, Bearish Wedge

---

## How It Works

1. **`OnInit()`** — normalizes `point` for 5-digit brokers; initializes `emaHandle` and `atrHandle` via `iMA()` and `iATR()`; returns `INIT_FAILED` if either handle is `INVALID_HANDLE`
2. **`OnTick()`** — compares `iTime(NULL, 0, 0)` to `lastBarTime`; skips if same bar. If positions exist, calls `ManageTrades()` and returns. Otherwise calls `DetectPatterns()`
3. **`DetectPatterns()`** — iterates `MaxBarsForPattern` bars, evaluating each registered pattern detection function and returning the first matched `PATTERN_TYPE`
4. **Signal validation** — `IsBullishPattern()` / `IsBearishPattern()` switch classifies the detected pattern; `IsBullishTrend()` / `IsBearishTrend()` optionally gate the signal against EMA direction; `UseVolumeFilter` gates against volume confirmation within individual detection functions
5. **`OpenBuy()` / `OpenSell()`** — fetch current ATR value from `atrHandle`, compute SL and TP in points, call `OrderSend()` with `OP_BUY` or `OP_SELL`, `LotSize`, `Slippage`, and `MagicNumber`
6. **`ManageTrades()`** — iterates open orders matching symbol and magic number; updates trailing stop distance based on current ATR when `UseTrailingStop = true`

---

## Input Parameters

| Parameter           | Type   | Default  | Description                                                        |
|---------------------|--------|----------|--------------------------------------------------------------------|
| `LotSize`           | double | `0.1`    | Trade lot size for all orders                                      |
| `MagicNumber`       | int    | `12345`  | Unique EA identifier applied to all orders                         |
| `Slippage`          | int    | `3`      | Maximum allowed slippage in points on order execution              |
| `UseTrailingStop`   | bool   | `true`   | Enable ATR-based trailing stop management on open positions        |
| `UseTrendFilter`    | bool   | `true`   | Gate entries against EMA trend direction                           |
| `EMAPeriod`         | int    | `50`     | Period for the EMA trend filter                                    |
| `UseVolumeFilter`   | bool   | `true`   | Enable volume confirmation within pattern detection functions      |
| `ATRPeriod`         | int    | `14`     | ATR period for dynamic stop loss and trailing stop calculation     |
| `RiskRewardRatio`   | double | `2.0`    | Take profit multiplier relative to stop loss distance              |
| `MaxBarsForPattern` | int    | `50`     | Maximum historical bars scanned per `DetectPatterns()` call        |

---

## Installation

1. Rename `New_Prophet_MegaGen_Xmk1.txt` → `New_Prophet_MegaGen_Xmk1.mq4`
2. Copy to the MetaTrader 4 **Experts** folder:
   ```
   MQL4/Experts/
   ```
3. Open MetaEditor and **compile** (F7) — resolve any `INVALID_HANDLE` warnings for your MT4 build
4. In MT4, open the **Navigator** panel → **Expert Advisors** → drag onto a chart
5. In the EA properties dialog, enable **"Allow live trading"** and configure inputs
6. Confirm the EA smiley face icon appears in the top-right corner of the chart

> **Warning:** This EA places real trades on a live account. Always run on a **demo account** first and forward-test for a minimum of several weeks before live deployment.

> **Note:** Ensure **AutoTrading** is enabled in the MT4 toolbar (the green play button).

---

## Requirements

- MetaTrader 4 (build supporting `OnInit` / `OnTick` / indicator handles)
- MQL4 compiler (MetaEditor)
- AutoTrading enabled in MT4

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
