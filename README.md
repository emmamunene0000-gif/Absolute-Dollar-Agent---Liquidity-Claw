# ABSOLUTE DOLLAR AGENT — LIQUIDITY CLAW
## ADSA v8.1 · The Claw Protocol Edition
*Augmented Intelligence Execution System — Pine Script v6 Strategy*

> **This is not an indicator. It is the codification of a real trader's mind.**

---

## THE ARCHITECT'S INTENT

Most trading tools measure price. ADSA does something different: it **augments how a trader thinks about price** — extracting intelligence from structure, session, liquidity, and momentum — and converts that intelligence into executable confidence with a defined risk-reward model.

Think of it as a **Jarvis for intraday traders**:

- It reads the market the way a professional trader reads it — session context, daily bias, structure, liquidity sweeps, momentum alignment
- It scores that reading against a multi-layer framework — disagreement between layers = low confidence = silence
- It executes when confidence is sufficient — firing to MT5 via TradeSgnl EA with exact levels calculated
- It coaches simultaneously — broadcasting Telegram smart cards that show not just what it did but **why**, at the moment it acts

The result: a mechanically discretionary execution module. It enforces the architect's framework without emotion, without hesitation, without deviation — and it teaches through every signal.

> **"Not a mix of indicators. An augmentation of a trader's process for extracting liquidity from the markets with confidence."**

---

## TABLE OF CONTENTS

1. [What This Agent Does](#1-what-this-agent-does)
2. [System Architecture](#2-system-architecture)
3. [Installation](#3-installation)
4. [Market Coverage](#4-market-coverage)
5. [The Five-Layer Signal Engine](#5-the-five-layer-signal-engine)
6. [The Claw Protocol — Liquidity Trail](#6-the-claw-protocol--liquidity-trail)
7. [Confidence Engine](#7-confidence-engine)
8. [Risk Management](#8-risk-management)
9. [MT5 Execution via TradeSgnl](#9-mt5-execution-via-tradesgnl)
10. [Trade Automation — BE, Trail, Partial Close](#10-trade-automation--be-trail-partial-close)
11. [Glass Box Broadcast — Telegram Smart Cards](#11-glass-box-broadcast--telegram-smart-cards)
12. [Timezone & Session Setup](#12-timezone--session-setup)
13. [Dashboard Guide](#13-dashboard-guide)
14. [Pyramiding Protocol](#14-pyramiding-protocol)
15. [Backtesting Methodology](#15-backtesting-methodology)
16. [Live Trading Checklist](#16-live-trading-checklist)
17. [Tuning Cheat Sheet by Market](#17-tuning-cheat-sheet-by-market)
18. [Known Behaviours & Edge Cases](#18-known-behaviours--edge-cases)

---

## 1. WHAT THIS AGENT DOES

ADSA v8.1 is a **real trader's execution framework codified into Pine Script v6**. It is not a standalone indicator — it is a full-stack intelligence and execution system that:

- Reads the market across **5 layers** simultaneously: macro trend, swing structure, momentum, execution pulse, and liquidity context
- Maintains a **live intelligence narrative** — scoring every bar against the confluence framework and narrating the trade rationale in plain language
- Detects **liquidity extraction zones** — swing highs/lows, PDH/PDL, order blocks, and resting buy/sell-side pools
- Uses a **ratcheting ATR trail** (The Claw Protocol) as the single hard directional gate — everything else is a weight
- Fires **TradeSgnl EA alerts** to MT5 with exact price levels, dollar risk, and partial close percentages — the EA calculates lots server-side
- Automates **break-even and trailing stop management** after entry, via a second alert channel, requiring no Python or manual modification
- Tracks every trade in a **performance array** with daily reset — giving accurate real-time statistics on win rate, profit factor, and pip expectancy
- **Broadcasts Telegram smart cards** — structured, non-repetitive event cards that expose the agent's full reasoning at every decision point: entries, exits, structure shifts, session opens, scale-ins, liquidity sweeps

The philosophy: **execute with confidence, not frequency**. Every signal must clear the confidence threshold. Below threshold = silence. Above threshold = fire with precision.

---

## 2. SYSTEM ARCHITECTURE

```
MARKET PRICE
    ↓
5-LAYER INTELLIGENCE READ (score: -15 to +15)
    ├── L1 Sovereign (1D/4H) — Macro trend
    ├── L2 Anchor (1H)       — Swing structure
    ├── L3 Filter (15m)      — Momentum direction
    ├── L4 Execution (TF)    — Local pulse
    └── L5 Liquidity         — Context & regime

    ↓
CLAW PROTOCOL (hard directional gate)
    └── MTF trail: Exec + M5 + M15 + H1 must agree

    ↓
CONFIDENCE ENGINE (weighted threshold test)
    └── Conservative / Moderate / Aggressive mode

    ↓
SIGNAL CONFIRMED (buy_signal_confirmed / sell_signal_confirmed)

    ┌───────────────────┬───────────────────────────┐
    ↓                   ↓                           ↓
STRATEGY ENTRY      TELEGRAM SMART CARD        TELEGRAM PUBLIC
(TradeSgnl alert)   (War Room — full Glass Box) (coaching channel)

    ↓
MT5 EA EXECUTION
    ├── Partial close at TP1 (33%) → auto-BE
    ├── Partial close at TP2 (33%) → trail activated
    └── Final close at TP3 (34%) or VWAP holder exit
```

### Signal Types

| Type | Trigger | Use case |
|------|---------|---------|
| **ATM Signal** | RSI regime + ATR crossover + Stoch + Claw | Core intraday signals |
| **Smart RSI** | RSI ≥ 75 / ≤ 25 extreme + Claw + Regime | High-momentum continuation |
| **Pyramid** | Smart RSI re-hit OR ATR cross + Claw, while base trade running in profit | Scale-in on winning trades |

All three signal types pass through the same Confidence Engine and fire through the same TradeSgnl execution channel.

---

## 3. INSTALLATION

### Step 1 — Copy the script
1. Open `Absolute Dollar Agent.txt` in this repo
2. Select all → Copy
3. In TradingView: Pine Editor → New Script → Paste → Save as **"ADSA v8.1 - Claw"**

### Step 2 — Add to chart
- Add as a **Strategy** (not indicator) — the Strategy Tester tab will appear
- Set chart to your execution timeframe (M1 for prop firm scalping, M5/M15 for swing scalps)

### Step 3 — Set your TradeSgnl License ID
- Script settings → `[⚙️ TradeSgnl EA]` group → paste your License ID
- Set your symbol exactly as it appears on MT5 (e.g. `EURUSD`, `Volatility 75 Index`, `XAUUSD`)

### Step 4 — Configure risk
- `[💰 Risk Management]` group → set `Risk Per Trade ($)` to your target dollar risk per trade
- Set your account balance (used for position sizing reference — actual lots calculated by the EA)

### Step 5 — Set timezone
- `[📊 Dashboard]` group → `UTC Offset` = **3** for Nairobi/EAT
- `Timezone Label` = **EAT**

### Step 6 — Create TradingView Alerts (TWO required)

**Alert 1 — Order Execution (TradeSgnl entries/exits)**
1. Right-click chart → Add Alert
2. Condition: **ADSA v8.1** → **Order fills only**
3. Message: `{{strategy.order.alert_message}}`
4. Webhook URL: your TradeSgnl webhook endpoint
5. Fire: **Once Per Bar Close**

**Alert 2 — BE/Trail Modifications**
1. Right-click chart → Add Alert
2. Condition: **ADSA v8.1** → **Any alert() function call**
3. Message: `{{alert_message}}`
4. Webhook URL: **same** TradeSgnl webhook endpoint
5. Fire: **Once Per Bar Close**

> Both alerts point to the same TradeSgnl webhook. Alert 1 handles entry and TP partial closes. Alert 2 handles break-even and trail modifications after TP1/TP2.

---

## 4. MARKET COVERAGE

| Market | Symbol Format | Notes |
|--------|--------------|-------|
| Forex Majors | `EURUSD`, `GBPUSD`, `XAUUSD` | Standard 5-digit pricing |
| Deriv Synthetics | `Volatility 75 Index`, `Crash 1000 Index`, `Boom 1000 Index` | Use `vol_dollar` = risk amount; EA handles lot math |
| Deriv Step Index | `Step Index` | Very tight spreads, good for trail testing |
| CFDs (Indices) | `US30`, `NAS100`, `GER40` | Larger ATR — increase TP multipliers |
| Crypto Perps | `BTCUSDT`, `ETHUSDT` | Bybit/Bitget perpetual syntax via TradeSgnl |
| Metals | `XAUUSD`, `XAGUSD` | ATR-based SL essential — never use fixed pip SL |

> **Deriv Synthetics Note**: v8.0+ sends `vol_dollar = risk_per_trade` directly to the EA. The MT5 EA knows the contract spec and calculates correct lots server-side. Never manually set lots on Deriv synthetics.

---

## 5. THE FIVE-LAYER SIGNAL ENGINE

The score runs from **-15 to +15**. Each layer adds or subtracts weight based on directional alignment.

### Layer 1 — Sovereign Trend (±3)
**Timeframe**: 4H or 1D (configurable as `sovereign_tf`)
- EMA 9 > EMA 21 = +3 bullish | EMA 9 < EMA 21 = -3 bearish
- **Purpose**: Prevents trading against the dominant wave. Counter-trend longs in a daily downtrend have lower structural probability.

### Layer 2 — Anchor Structure (±3)
**Timeframe**: 1H (configurable as `anchor_tf`)
- EMA alignment on the anchor TF = ±3
- **Purpose**: The swing trading layer — identifies intermediate structure you are trading with

### Layer 3 — Filter Momentum (±2)
**Timeframe**: 15m (configurable as `filter_tf`)
- MACD crossover state combined with EMA stack
- **Purpose**: Filters counter-trend scalps on the execution TF

### Layer 4 — Execution Pulse (±2)
**Timeframe**: Current chart TF
- RSI position relative to 50, Stochastic state, volume MA confirmation
- **Purpose**: Local momentum confirmation — is this bar ready?

### Layer 5 — Liquidity Context (±5 total)
- PDH/PDL proximity (+1 each)
- Swing high/low break confirmation (+1 each)
- ATR regime: High ATR = volatility present = +1

### Reading the Score

| Score | Label | Action |
|-------|-------|--------|
| ≥ 10 + High ATR | 🔥 SOVEREIGN HIGH MOMENTUM | Maximum conviction — size up |
| ≥ 6 | 📈 BULLISH BIAS (Strong) | Take setups, size normally |
| 3–5 | 📈 BULLISH BIAS (Moderate) | Take setups, size conservatively |
| 1–2 | ↗ Bullish Lean | Wait for Claw confirmation before entry |
| -2 to +2 | ⚪ NEUTRAL | Stay flat — no edge |
| -3 to -5 | 📉 BEARISH LEAN/MODERATE | Mirror logic of bullish |
| ≤ -10 + High ATR | 🥶 SOVEREIGN HIGH MOMENTUM BEARISH | Maximum conviction short |

---

## 6. THE CLAW PROTOCOL — LIQUIDITY TRAIL

The Claw is the **single hard gate** in the system. A signal does not fire unless The Claw agrees with the direction. Every other layer (VWAP, Fib Trend, Score) is a weight — The Claw is a gate.

### How calcLiqTrail() Works

```
1. Compute EMA(close, ma_length)             → centre line
2. Compute ATR(atr_length)                   → volatility band
3. Bull trail = EMA - ATR × multiplier       → trails below price
4. Bear trail = EMA + ATR × multiplier       → trails above price
5. RATCHET: uptrend trail only moves UP (math.max)
            downtrend trail only moves DOWN (math.min)
6. FLIP: When price crosses through trail → trend flips, trail resets
```

The ratcheting mechanism means the trail **never gives back distance** in the trend direction. It tightens as momentum builds. It is the agent's single source of directional truth.

### Multi-Timeframe Claw Stack

| Instance | TF | Purpose |
|---------|----|---------|
| `claw_exec` | Chart TF | Entry timing gate |
| `claw_m5` | 5m | Short-term direction truth |
| `claw_m15` | 15m | Intraday structure direction |
| `claw_h1` | 1H | Swing direction filter |

**MTF Alignment rule**: `claw_mtf_bull = claw_m15 UP AND claw_h1 UP` — both must agree for full confidence score. If only one agrees, the confidence score is penalised.

### Dashboard Claw Display
```
🦞 Claw 🟢  M5:▲  M15:▲  H1:▲   ← All green = maximum directional confidence
🦞 Claw 🔴  M5:▼  M15:▲  H1:▲   ← Exec TF against structure = wait
```

---

## 7. CONFIDENCE ENGINE

The Confidence Engine takes the raw signal and decides **whether to fire** based on a weighted percentage score.

### Weights

| Component | Weight |
|-----------|--------|
| Claw MTF full alignment | +3 |
| Score ≥ threshold | +2 |
| VWAP swing (if enabled) | +2 |
| Fib Trend (if enabled) | +1 |
| ATR regime = High | +1 |
| Smart RSI extreme hit | +2 |

**Maximum confidence**: ~11 points

### Thresholds by Mode

| Mode | Threshold | Best for |
|------|-----------|---------|
| Conservative | 60% | Live trading, learning phase |
| Moderate | 45% | Backtested setups with positive EV |
| Aggressive | 30% | High-volume synthetics, experienced traders |

> Start Conservative. Run 100 backtested trades. If win rate is above 55% at Conservative, shift to Moderate. Never go Aggressive on a new market.

---

## 8. RISK MANAGEMENT

### The Dollar Risk Model

ADSA v8.1 uses **fixed dollar risk** per trade, not a fixed lot size. This is correct for:
- Deriv synthetics (variable contract specs)
- Cross-asset portfolios (Gold, Forex, Crypto, Indices — all from one $ risk input)
- Accounts of any size

**Formula**: `Risk Per Trade ($)` input → sent as `vol_dollar` to TradeSgnl EA → EA calculates lots using contract specs server-side.

### SL and TP Placement (ATR-based)

| Level | Multiplier | Action |
|-------|-----------|--------|
| Stop Loss | ATR × `sl_buffer_atr` | Beyond volatility noise |
| TP1 (1:1) | ATR × 1.0× | Triggers auto-BE + 33% partial close |
| TP2 (1.5:1) | ATR × 1.5× | Triggers trail + 33% partial close |
| TP3 (2:1) | ATR × 2.0× | Activates holder mode, 34% close |

### Three-Part Partial Close

Every entry fires with `pct1=0.33, pct2=0.33, pct3=0.34` — splitting the position into three equal-thirds. This means:
- TP1 hit → 33% booked. SL moved to break-even. Risk eliminated.
- TP2 hit → another 33% booked. Trail activated. Position running for free.
- TP3 hit → final 34% booked. Holder mode activates VWAP trail on remaining.

---

## 9. MT5 EXECUTION VIA TRADESGNL

TradeSgnl is an MT5 EA that listens to TradingView alerts and executes trades. ADSA v8.1 controls all logic; the EA blindly executes the instructions.

### Entry Alert Message Format

```
LICENSE_ID,XAUUSD,sell,vol_dollar=15,sl_price=4080.20,tp1_price=4062.12,pct1=0.33,tp2_price=4057.59,pct2=0.33,tp3_price=4053.07,pct3=0.34,exent=1
```

| Field | Value | Meaning |
|-------|-------|---------|
| `vol_dollar` | Dollar risk | EA calculates lots from this |
| `sl_price` | Exact SL price | Entry ± ATR×buffer |
| `tp1_price` | 1:1 target | 33% closed here |
| `pct1` | 0.33 | 33% at TP1 |
| `tp2_price` | 1.5:1 target | 33% closed here |
| `pct2` | 0.33 | 33% at TP2 |
| `tp3_price` | 2:1 target | 34% closed here |
| `pct3` | 0.34 | Final 34% at TP3 |
| `exent` | 1 | Execute immediately; close opposite if open |

### Pyramid Scale-In Alert Format

```
LICENSE_ID,XAUUSD,sell,vol_dollar=7.5,sl_price=4080.20,tp1_price=4062.12,pct1=0.33,tp2_price=4057.59,pct2=0.33,tp3_price=4053.07,pct3=0.34,exent=1
```

Scale-in alerts fire through the same channel with reduced `vol_dollar` (configurable `pyramid_risk_pct` of base risk).

---

## 10. TRADE AUTOMATION — BE, TRAIL, PARTIAL CLOSE

After entry, the EA needs modify instructions to manage the trade. These fire via **Alert 2** (the `alert()` function call channel).

### Break-Even on TP1

When price hits TP1:
```
LICENSE_ID,XAUUSD,modifysell,sl_price=4071.16
```
SL moves to entry + buffer. Trade is risk-free from this point.

### Trailing Stop on TP2

When price hits TP2:
```
LICENSE_ID,XAUUSD,modifysell,trail=1,trail_start=0.0032,trail_step=0.0016
```
`trail_start` = 1.5× current ATR. `trail_step` = 0.5× ATR. EA manages trail server-side.

### Enabling Automation

In `[⚙️ TradeSgnl EA]` group:
- `✅ Auto BE on TP1` — move SL to break-even when TP1 hit
- `✅ Auto Trail on TP2` — activate EA trail when TP2 hit
- `BE Buffer Points` — points above/below entry to set the BE stop

---

## 11. GLASS BOX BROADCAST — TELEGRAM SMART CARDS

The agent broadcasts two Telegram channels: **War Room** (premium, full Glass Box) and **Public** (sanitized coaching feed). Every event gets its own self-contained smart card — no repeated data, no filler.

### Entry Card (War Room)

```
━━━━━━━━━━━━━━━━━━━━━
🥶 ABSOLUTE DOLLAR — MASTER SYNC SHORT
XAUUSD | M1 | 4071.16 | 👑 LONDON | 10:02 EAT
━━━━━━━━━━━━━━━━━━━━━
📋 EXECUTION PLAN
🔴 SHORT @ 4071.16
🛑 SL   → 4080.20  (-9.0 pts)
🎯 TP1  → 4062.12  (1:1)
🎯 TP2  → 4057.59  (1.5:1)
🚀 TP3  → 4053.07  (2:1)
💰 $15 risk | 0.03 lots
─────────────────────
⚡ WHY NOW
🥶 FULLY ALIGNED: BEARISH (4-LAYER)
Score: -13/15 | 🥶 SOVEREIGN HIGH MOMENTUM BEARISH
Daily: Inside Range [PDH: 4095.00 | PDL: 4030.00]
Claw:  🔴 MTF BEAR
─────────────────────
🏗 5-LAYER READ
📉 BEARISH BIAS (Strong) [-13/15]
├─ L1 Sovereign: Daily EMA9 < EMA21 ✓ BEAR
├─ L2 Anchor: H1 structure bearish ✓ BEAR
├─ L3 Filter: 15m MACD declining ✓ BEAR
├─ L4 Exec: RSI below 50, Stoch bear ✓ BEAR
└─ Liquidity: PDL at 4030, sell pool cleared ✓ BEAR
─────────────────────
All layers aligned. 4-layer execution window active.
PF: 2.31  WR: 63.6%
🏷 ATM-20260611-1002-SELL-1
━━━━━━━━━━━━━━━━━━━━━
⚠️ Not financial advice. © 2026 Absolute Dollar
```

### Session Battle Plan Card

```
━━━━━━━━━━━━━━━━━━━━━
👑 LONDON SESSION — BATTLE PLAN
XAUUSD | M1 | 4071.16 | 10:00 EAT
━━━━━━━━━━━━━━━━━━━━━
📋 SESSION HYPOTHESIS
Bias:  🥶 SOVEREIGN HIGH MOMENTUM BEARISH
Score: -13/15 | Sovereign: BEAR
Daily: Inside Range [PDH: 4095 | PDL: 4030]
ATR:   High — Expansion active. Setups live.
Agent: HUNTING SHORT SETUPS
─────────────────────
⚡ SIGNAL STATUS
Sync:  🥶 FULLY ALIGNED: BEARISH (4-LAYER)
Claw:  🔴 MTF BEAR
Today: 2 sigs | PF: 2.31
─────────────────────
🎯 KEY LEVELS
🔴 Sell-side liq: 4030.00
🟢 Buy-side liq:  4095.00
Context: 💧 PREMIUM ZONE
━━━━━━━━━━━━━━━━━━━━━
⚠️ Not financial advice. © 2026 Absolute Dollar
```

### Scale-In Card

```
━━━━━━━━━━━━━━━━━━━━━
🔺 SCALE-IN LEG 1 — ADDING TO SHORT
XAUUSD | M1 | Winner running
━━━━━━━━━━━━━━━━━━━━━
📋 PYRAMID PLAN
🔴 SHORT @ 4065.00
🛑 SL   → 4072.00
🚀 TP3  → 4051.00  (2:1)
💰 $7.5 (50.0% base)
─────────────────────
Trigger: RSI extreme + 🔴 MTF BEAR
Base trade: +6.16 pts unrealised
Agent tightening base trail simultaneously.
━━━━━━━━━━━━━━━━━━━━━
⚠️ Not financial advice. © 2026 Absolute Dollar
```

### TP1 Card

```
━━━━━━━━━━━━━━━━━━━━━
🎯 TP1 CLEARED — LEG 1 SECURED (1:1)
XAUUSD | SHORT from 4071.16
━━━━━━━━━━━━━━━━━━━━━
✅ 33% taken at 4062.12
🔐 Breakeven set → 4071.16
🎯 Hunting TP2 → 4057.59  (1.5:1)
─────────────────────
Agent sending BE modify to MT5.
━━━━━━━━━━━━━━━━━━━━━
⚠️ Not financial advice. © 2026 Absolute Dollar
```

### Stop Loss Autopsy Card

```
━━━━━━━━━━━━━━━━━━━━━
💀 STOP ABSORBED — GLASS BOX AUTOPSY
XAUUSD | SHORT | ATM-20260611-1002-SELL-1
━━━━━━━━━━━━━━━━━━━━━
🔬 MACRO ROTATION: Higher-TF structure shifted
post-entry. Trade was structurally valid —
context rotated against us.
─────────────────────
Peak before stop: 3.40 pts
Score at entry:   -13.0/15
Risk absorbed:    ~$14.85
─────────────────────
Every loss is data. Reassessing.
━━━━━━━━━━━━━━━━━━━━━
⚠️ Not financial advice. © 2026 Absolute Dollar
```

### Configuring Telegram

In `[📡 Telegram Broadcast]` settings:
- `War Room Chat ID` — premium channel ID (or group ID)
- `Public Channel ID` — free coaching channel
- `✅ Sanitize Public Channel` — hides exact price levels from public feed
- `✅ Enable Agent Alerts` — master toggle

---

## 12. TIMEZONE & SESSION SETUP

| Session | UTC Open | EAT Open | Character |
|---------|---------|---------|-----------|
| Tokyo / Asia | 00:00 | 03:00 | Range, low liquidity |
| London Open | 07:00 | **10:00** | 🔑 First major move of the day |
| NY Open | 13:00 | **16:00** | 🔑 Highest liquidity |
| London Close | 16:00 | 19:00 | Fakeouts common — be cautious |
| NY Close | 22:00 | 01:00 | No new trades |

### Prime Windows from Nairobi (EAT)
```
09:30–10:30 EAT  → Pre-London (liquidity sweeps before open)
10:00–12:00 EAT  → London open — ADSA prime hunting window
14:00–16:00 EAT  → London/NY overlap — momentum extension
16:00–18:00 EAT  → NY open — second wave
```

### Configuring Timezone
- `[📊 Dashboard]` → `UTC Offset` = **3** → `Timezone Label` = **EAT**

---

## 13. DASHBOARD GUIDE

The ADSA cognitive dashboard is a live trading plan — readable in one glance.

```
══════════════════════════════════════
 ⏱️ FRACTAL 4-LAYER SYNC
 L1 Sovereign (1D): BEAR
 L2 Anchor   (1H): BEAR
 L3 Filter  (15m): BEAR
 L4 Exec     (M1): BEAR
 🥶 FULLY ALIGNED: BEARISH

 📊 CORE SIGNALS
 XAUUSD | M1  4071.16
 👑 LONDON SESSION (10:xx EAT) | ATR: High (9.05)
 📅 Daily: Inside Range [PDH: 4095 | PDL: 4030]
 🦞 Claw Trail: 🔴 MTF BEAR
 🎯 Confidence: 73% (need 60%)
 ATM 🔴  Regime 📉
 VWAP 📉  Fib 📉
 RSI 📉 (38)  MACD: 📉

 🦞 Claw 🔴  M5:▼  M15:▼  H1:▼

 🧠 5-LAYER AI NARRATIVE [-13/15]
 🥶 SOVEREIGN HIGH MOMENTUM BEARISH
 ├─ L1 Sovereign: Daily EMA9 < EMA21 ✓
 ├─ L2 Anchor: H1 structure bearish ✓
 ├─ L3 Filter: 15m MACD declining ✓
 ├─ L4 Exec: RSI below 50, Stoch bear ✓
 └─ Liquidity: Sell pool 4030, PDL below ✓

 💡 AGENT ADVICE
 "All layers aligned. Claw confirms. Active sell zone.
  Aggressive entry — 4-layer sovereign aligned."

 📊 TODAY
 Sigs: 2  🔺Pyr: 0  Blocked: 4
 LDN: 2  NY: 0  ASIA: 0
══════════════════════════════════════
```

---

## 14. PYRAMIDING PROTOCOL

Pyramiding in ADSA v8.1 means **adding to a confirmed winning position** — not doubling down on a loser.

### Pyramid Engine Triggers

A scale-in fires when ALL of the following are true:
1. A base trade is already running (`is_trade_running = true`)
2. The base trade is in **minimum profit** (configurable `pyramid_min_profit` pips)
3. A **new Smart RSI extreme** fires (RSI re-hits ≥75/≤25 while in trend) OR a fresh ATR crossover + Claw alignment
4. The cooldown bar count since last leg has passed (`pyramid_cooldown` bars)
5. Maximum pyramid legs not yet reached (`max_pyramids` = 2 default)

### Pyramid Risk Rule

```
Pyramid lot = Base risk × pyramid_risk_pct (default 50%)
```

Never pyramid at full size. The pyramid adds to conviction at reduced risk.

### Pyramid Inputs

In `[🔺 Pyramiding / Scale-In]` settings:
- `✅ Enable Scale-In` — master toggle
- `Max Pyramid Legs` — 1–4, default 2
- `Pyramid Risk % of Base` — risk as % of base trade dollar risk
- `Min Profit to Scale (pips)` — base trade must be in this much profit before scaling
- `Cooldown (bars between legs)` — prevents stacking too quickly

### When NOT to Pyramid
- Base trade has NOT yet hit TP1 (still at risk)
- Claw has flipped direction at any MTF
- ATR is in Low regime (volatility dried up)
- Approaching session close without NY catalyst

---

## 15. BACKTESTING METHODOLOGY

### The Right Approach

Pine Script's Strategy Tester gives **historical performance data** but requires careful interpretation for intraday strategies.

#### Step 1 — Baseline Test
1. Set chart to execution TF, 500+ bars
2. `Commission`: match your broker (0.0002 for Forex, 0.1% for crypto)
3. `Slippage`: 1–2 ticks Forex, 3–5 synthetics
4. Run Conservative mode (60%)
5. Record: Win rate, Profit Factor, Max Drawdown

#### Step 2 — What Good Numbers Look Like

| Metric | Minimum | Target | Excellent |
|--------|---------|--------|-----------|
| Win Rate | 45% | 55% | 65%+ |
| Profit Factor | 1.3 | 1.8 | 2.5+ |
| Max Drawdown | < 20% | < 12% | < 8% |
| Trades / week | — | 3–7 | — |

> **Sample Size Warning**: Fewer than 100 trades is not a meaningful backtest. A 2.3 PF over 11 trades can be random luck. Target 100+ trades over 3+ months before trusting a filter setting.

#### Step 3 — Walk-Forward Validation
- Backtest Jan–Sep
- Forward-test Oct–Dec (don't touch settings)
- If performance degrades > 30%, one layer is misfiring — investigate which one

#### Tester Trust Level
The Strategy Tester is **directionally reliable** for signal quality. It is **NOT reliable** for exact P&L (uses close prices, not tick fills). Treat it as conservative vs. live execution — if the tester is profitable, live execution should be at least as good.

---

## 16. LIVE TRADING CHECKLIST

### Pre-Session (before London open = before 10:00 EAT)
```
□ Check 1D chart — what is the daily bias?
□ Mark PDH and PDL
□ Mark most recent swing high and low
□ Is ATR High or Low? (Low ATR = no trades)
□ Is sovereign TF aligned with intended direction?
□ Check economic calendar — high-impact news in next 2 hours?
□ TradeSgnl EA running on MT5 with license active
□ Both TradingView alerts pointing to webhook
□ Telegram receiving session battle plan card
```

### During Session
```
□ Only take signals during London (10:00–12:00 EAT) or NY (16:00–18:00 EAT)
□ Confidence above threshold
□ Claw aligned with direction — no neutral Claw entries
□ All 4 MTF EMA layers preferred — minimum 3 of 4
□ Counter-trend signals (sovereign_counter): TP1 only, no TP3
□ Maximum 2 open positions simultaneously
□ If Telegram shows ⛔ SIGNAL BLOCKED — respect the filter
```

### Post-Session
```
□ Wait for daily report (fires at configured UTC hour)
□ Review: Did every trade match signal conditions?
□ Document missed signals and false signals
□ Note which layer misfired on losing trades
```

---

## 17. TUNING CHEAT SHEET BY MARKET

### XAUUSD (Gold) — M1/M5
```
Execution TF:     M1 (prop firm) or M5 (standard)
ATR Length:       14
ATR Multiplier:   1.5–2.0
Confidence mode:  Moderate (45%)
Best sessions:    London 10:00 EAT, NY 16:00 EAT
Notes: High ATR = large SL. Keep risk_per_trade conservative.
       Respect PDH/PDL hard. Enable Fib gate in trending sessions.
```

### Deriv Volatility 75 Index — M1/M5
```
Execution TF:     M1 or M5
ATR Length:       10
ATR Multiplier:   2.0 (fast rejection wicks)
Confidence mode:  Moderate (45%)
Best sessions:    24/7 — follow NY session for trending behaviour
Notes: vol_dollar essential — lots are non-standard.
       Never manually set lots on V75.
```

### EURUSD / GBPUSD — M15
```
Execution TF:     M15
ATR Length:       14
ATR Multiplier:   1.5
Confidence mode:  Conservative (60%)
Best sessions:    London and NY open only
Notes: Enable VWAP filter. Avoids false signals in ranging sessions.
```

### US30 / NAS100 — M5
```
Execution TF:     M5
ATR Length:       14
ATR Multiplier:   1.2 (indices move fast)
Confidence mode:  Moderate (45%)
Best sessions:    NY only (16:00–19:00 EAT)
Notes: Check futures gap at NY open. If gap > 50 pts,
       wait for first M5 candle to close before trading.
```

### Deriv Crash / Boom 1000 — M1
```
Execution TF:     M1
Trade direction:  Crash = SELL only. Boom = BUY only.
ATR Multiplier:   3.0 (spike events need room)
Confidence mode:  Aggressive (30%)
Notes: Disable VWAP filter. Disable Fib Trend.
       Use Claw direction as primary gate.
       Never trade against the synthetic trend direction.
```

---

## 18. KNOWN BEHAVIOURS & EDGE CASES

### Bar Replay vs Live
ADSA uses `barstate.isconfirmed` to prevent triggers mid-bar. In live trading, signals fire **on bar close** — not within the bar. This prevents false entries from wicks.

### MTF Claw Repainting
The MTF Claw values use `barstate.isconfirmed` inside `request.security()`. This eliminates repainting on live bars. Historical bars show the confirmed close value only.

### posState Gate
`buy_signal_confirmed` requires `posState[1] <= 0` — prevents same-direction re-entry while a position is already open. `posState` resets to 0 on exit (SL hit or holder mode close). If you see "Blocked: N" with "Sigs: 0" on the dashboard, check that exits are being registered (exit block requires `sl_alert_event` or `holder_exit_event` to fire).

### Regime Filter
- When `enableRegimeFilter = true`, signals only fire if RSI regime matches direction
- VWAP and Fib Trend are **independent weights** — each narrows valid regimes but they are not combined gates
- Fib gate ON dramatically improves PF in trending conditions (validated: PF 0.99 → 2.3 with gate ON on XAUUSD bear market). Valid as a trend filter — not overfitting.

### Counter-Trend Signals
If the sovereign layer disagrees with the signal direction (`sovereign_counter_buy/sell`):
- Signal fires but Telegram displays a **counter-trend warning**
- Rule: Target TP1 only. Do NOT hold to TP3.
- These signals have lower structural probability by design.

### Strategy Tester Limitations
- Strategy Tester cannot simulate partial closes perfectly — TP1/TP2 appear as separate exit orders
- `pct1/pct2/pct3` fields in TradeSgnl alerts handle actual partial close on MT5
- Use the tester for directional validation and signal quality. Not for exact P&L simulation.

---

## QUICK REFERENCE — TRADE DECISION TREE

```
┌──────────────────────────────────────────────┐
│  ADSA v8.1 — AGENT DECISION FRAMEWORK        │
│                                              │
│  1. ATR = HIGH?           No → Skip          │
│  2. Claw aligned?         No → Skip          │
│  3. Score ≥ 6?            No → Wait          │
│  4. Confidence ≥ 60%?     No → Skip          │
│  5. Session active?       No → Skip          │
│  6. SIGNAL FIRES → MT5 entry + Telegram card │
│                                              │
│  TP1 hit → 33% close + auto-BE               │
│  TP2 hit → 33% close + trail activated       │
│  TP3 hit → 34% close + VWAP holder mode      │
│  VWAP cross → close remaining position       │
│                                              │
│  PYRAMID: base in profit + fresh trigger     │
│  → scale-in at 50% base risk                 │
└──────────────────────────────────────────────┘
```

---

## VERSION HISTORY

| Version | Key Changes |
|---------|------------|
| v7.0 | Original SUPREME AGENT — ATM + VWAP + regime engine |
| v8.0 | + Claw Protocol (ratcheting ATR trail, MTF stack) |
| v8.0 | + Confidence Engine (Conservative / Moderate / Aggressive) |
| v8.0 | + Smart RSI extreme signals (75+/25− momentum entries) |
| v8.0 | + TradeSgnl BE/Trail automation (no Python required) |
| v8.0 | + Orphan trade protection (`dpt[]` array always accurate) |
| v8.0 | + `vol_dollar` for Deriv synthetics |
| v8.0 | + `regimeBullish/Bearish` CE10156 fix (sequential ternaries) |
| v8.1 | + Pyramid/Scale-In Engine (Section 16.5) |
| v8.1 | + `posState := 0` on exit (Sigs: 0 dashboard bug fixed) |
| v8.1 | + `pct3=0.34` added to base and pyramid entry messages |
| v8.1 | + Section 26 Glass Box Broadcast — smart card format, zero repetition |
| v8.1 | + README rewritten to reflect agent's true identity |

---

*ABSOLUTE DOLLAR INTELLIGENCE — Read the markets. Extract the liquidity. Execute with confidence.*
