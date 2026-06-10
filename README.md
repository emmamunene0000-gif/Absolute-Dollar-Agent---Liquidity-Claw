# ABSOLUTE DOLLAR AGENT — LIQUIDITY CLAW
## ADSA v8.0 · The Claw Protocol Edition
*Autonomous Decision Support System — Pine Script v6 Strategy*

> **Built for intraday liquidity extraction across Forex, Deriv Synthetics, and CFDs via MT5 + TradeSgnl EA handshake**

---

## TABLE OF CONTENTS

1. [What This Agent Does](#1-what-this-agent-does)
2. [Core Architecture](#2-core-architecture)
3. [Installation](#3-installation)
4. [Market Coverage](#4-market-coverage)
5. [The Five-Layer Signal Engine](#5-the-five-layer-signal-engine)
6. [The Claw Protocol — Liquidity Trail](#6-the-claw-protocol--liquidity-trail)
7. [Confidence Engine](#7-confidence-engine)
8. [Risk Management](#8-risk-management)
9. [MT5 Execution via TradeSgnl](#9-mt5-execution-via-tradesgnl)
10. [Trade Automation — BE, Trail, Partial Close](#10-trade-automation--be-trail-partial-close)
11. [Timezone & Session Setup (Nairobi/EAT)](#11-timezone--session-setup-nairobieat)
12. [Dashboard Guide](#12-dashboard-guide)
13. [Backtesting Methodology](#13-backtesting-methodology)
14. [Live Trading Checklist](#14-live-trading-checklist)
15. [Pyramiding Protocol](#15-pyramiding-protocol)
16. [Tuning Cheat Sheet by Market](#16-tuning-cheat-sheet-by-market)
17. [Known Behaviours & Edge Cases](#17-known-behaviours--edge-cases)

---

## 1. WHAT THIS AGENT DOES

ADSA v8.0 is a **complete intraday liquidity extraction system** built in Pine Script v6. It is not a standalone indicator — it is a full strategy that:

- Identifies **high-probability directional bias** using a five-layer confluence engine
- Measures **confidence as a weighted score** (like a trading plan pre-flight checklist)
- Detects **liquidity resting zones** (swing highs/lows, PDH/PDL, order blocks)
- Uses a **ratcheting ATR trail** (The Claw) as the directional truth gate
- Fires **TradeSgnl EA alerts** to execute on MT5 with correct lot sizing, SL, TP1, TP2
- Automates **break-even and trailing stop** management after entry without Python
- Tracks every trade in a **performance array** so the daily report is always accurate
- Displays a **cognitive dashboard** showing bias, session, confidence, regime, and AI narrative — all in your local timezone

The philosophy: **execute with confidence, not frequency**. Every signal must clear a confidence threshold. Below threshold = silence. Above threshold = fire.

---

## 2. CORE ARCHITECTURE

```
TRIGGER (ATM timing + RSI regime)
    ↓
GATE (Claw liquidity trail direction = hard gate)
    ↓
CONFLUENCE (5-layer weighted score)
    ↓
CONFIDENCE THRESHOLD (Conservative / Moderate / Aggressive)
    ↓
OPTIONAL FILTERS (VWAP swing, Fib Trend — each independently layered, not combined gates)
    ↓
EXECUTION (TradeSgnl alert → MT5 EA)
    ↓
AUTOMATION (BE on TP1 · Trail on TP2 · Track in dpt[] arrays)
```

### The Two Types of Signal

| Type | How it fires | When to use |
|------|-------------|-------------|
| **ATM signal** | RSI regime + ATR trail crossover + Stoch confirmation | Core intraday signals |
| **Smart RSI signal** | RSI hits extreme zone (≥75 bull / ≤25 bear) + Claw alignment + Regime | High-momentum entries, continuation plays |

Both signal types go through the same Confidence Engine and are subject to the same regime filters.

---

## 3. INSTALLATION

### Step 1 — Copy the script
1. Open `Absolute Dollar Agent.txt` in this repo
2. Select all → Copy
3. In TradingView: Pine Editor → New Script → Paste → Save as **"ADSA v8.0 - Claw"**

### Step 2 — Add to chart
- Add as a **Strategy** (not indicator) — the Strategy Tester tab will appear
- Set chart to your execution timeframe (recommended: **5m or 15m** for intraday)

### Step 3 — Set your TradeSgnl License ID
- In script settings → `[⚙️ TradeSgnl EA]` group → paste your License ID
- Set your symbol exactly as it appears on MT5 (e.g. `EURUSD`, `Volatility 75 Index`, `XAUUSD`)

### Step 4 — Configure risk
- `[💰 Risk Management]` group → set `Risk Per Trade ($)` to your target dollar risk per trade
- Set your account balance (used for position sizing reference only — actual lots are calculated by the EA)

### Step 5 — Set timezone
- `[📊 Dashboard]` group → `UTC Offset` = **3** for Nairobi/EAT
- `Timezone Label` = **EAT**

---

## 4. MARKET COVERAGE

| Market | Symbol Format | Notes |
|--------|--------------|-------|
| Forex Majors | `EURUSD`, `GBPUSD`, `XAUUSD` | Standard 5-digit pricing |
| Deriv Synthetics | `Volatility 75 Index`, `Crash 1000 Index`, `Boom 1000 Index` | Use `vol_dollar` = risk amount; EA handles lot math |
| Deriv Step Index | `Step Index` | Very tight spreads, good for trail testing |
| CFDs (Indices) | `US30`, `NAS100`, `GER40` | Larger ATR — increase TP multipliers |
| Crypto Perps | `BTCUSDT`, `ETHUSDT` | Bybit perpetual syntax supported via TradeSgnl |
| Metals | `XAUUSD`, `XAGUSD` | ATR-based SL essential — avoid fixed pip SL |

> **Deriv Synthetics Note**: The original lot sizing was calculating unrealistic sizes on synthetics because Pine Script cannot know contract specs. v8.0 fixes this by sending `vol_dollar = risk_per_trade` directly to the EA — the MT5 EA knows the contract and calculates correct lots server-side.

---

## 5. THE FIVE-LAYER SIGNAL ENGINE

The score runs from **-15 to +15**. Each layer adds or subtracts weight.

### Layer 1 — Sovereign Trend (±3)
**Timeframe**: 4H or 1D (configurable as `sovereign_tf`)
- EMA 9 > EMA 21 on the sovereign TF = +3 bullish weight
- EMA 9 < EMA 21 = -3 bearish weight

**Why it matters**: Prevents you from fighting the dominant wave. If the daily is in a downtrend, long scalps have lower probability of clean follow-through.

### Layer 2 — Anchor Structure (±3)
**Timeframe**: 1H (configurable as `anchor_tf`)
- EMA alignment on the anchor TF = ±3
- This is the **swing trading layer** — identifies the intermediate structure you are trading with

### Layer 3 — Filter Momentum (±2)
**Timeframe**: 15m (configurable as `filter_tf`)
- MACD crossover state combined with EMA stack
- Filters out counter-trend scalps on the execution TF

### Layer 4 — Execution Pulse (±2)
**Timeframe**: Current chart TF
- RSI position relative to mid (50), Stochastic state, and volume MA confirmation

### Layer 5 — Liquidity Context (±5 total)
- PDH/PDL proximity (+1 each)
- Swing high/low break confirmation (+1 each)
- ATR regime (High ATR = volatility present = +1)

### Reading the Score

| Score | Label | Meaning |
|-------|-------|---------|
| ≥ 10 AND High ATR | 🔥 SOVEREIGN HIGH MOMENTUM | Maximum conviction — all layers aligned with volatility |
| ≥ 6 | 📈 BULLISH BIAS (Strong) | Take setups, size normally |
| 3–5 | 📈 BULLISH BIAS (Moderate) | Take setups, size conservatively |
| 1–2 | ↗ Bullish Lean | Wait for Claw confirmation before entry |
| -2 to +2 | ⚪ NEUTRAL | Stay flat — no edge |
| -3 to -5 | 📉 BEARISH LEAN/MODERATE | Mirror logic of bullish |
| ≤ -10 AND High ATR | 🔥 SOVEREIGN HIGH MOMENTUM BEARISH | Maximum conviction short |

---

## 6. THE CLAW PROTOCOL — LIQUIDITY TRAIL

The Claw is the **only hard gate** in the system. A signal does not fire unless The Claw agrees with the direction. Everything else (VWAP, Fib Trend) is a weight — The Claw is a gate.

### How calcLiqTrail() Works

```
1. Compute EMA(close, ma_length) → centre line
2. Compute ATR(atr_length) → volatility band
3. Upper band = EMA - ATR × multiplier  (bull trail below price)
4. Lower band = EMA + ATR × multiplier  (bear trail above price)
5. RATCHET: In uptrend, trail only moves UP (math.max). In downtrend, trail only moves DOWN (math.min).
6. FLIP: When price crosses through the trail, trend flips and trail resets to the other band.
```

This ratcheting mechanism means the trail **never gives back distance** in the trend direction. It's a one-way ratchet that tightens as momentum builds.

### Multi-Timeframe Claw Stack

The agent runs four Claw instances simultaneously:

| Instance | TF | Purpose |
|---------|----|---------|
| `claw_exec` | Chart TF | Entry timing gate |
| `claw_m5` | 5m | Short-term direction truth |
| `claw_m15` | 15m | Intraday structure direction |
| `claw_h1` | 1H | Swing direction filter |

**MTF Alignment rule**: `claw_mtf_bull = claw_m15 UP AND claw_h1 UP` — both must agree for full confidence. If they disagree, confidence score is penalised.

### Claw Trail on the Dashboard
```
🦞 Claw 🟢  M5:▲  M15:▲  H1:▲   ← All green = maximum directional confidence
🦞 Claw 🔴  M5:▼  M15:▲  H1:▲   ← Exec TF against structure = wait
```

---

## 7. CONFIDENCE ENGINE

The Confidence Engine takes the raw signal and decides **whether to fire** based on a weighted score.

### Weights (configurable)

| Component | Bull weight | Bear weight |
|-----------|------------|------------|
| Claw MTF alignment | +3 | +3 |
| Score ≥ threshold | +2 | +2 |
| VWAP swing (if enabled) | +2 | +2 |
| Fib Trend (if enabled) | +1 | +1 |
| ATR regime = High | +1 | +1 |
| Smart RSI extreme | +2 | +2 |

**Maximum confidence**: ~11 points

### Confidence Thresholds by Mode

| Mode | Threshold | Best for |
|------|-----------|---------|
| Conservative | 60% | Live trading, learning phase |
| Moderate | 45% | Backtested setups with positive EV |
| Aggressive | 30% | High-volume synthetics, experienced traders |

> **Tuning tip**: Start Conservative. Run 100 backtested trades. If win rate is above 55% at Conservative, shift to Moderate. Never go Aggressive on a new market.

---

## 8. RISK MANAGEMENT

### The Dollar Risk Model

ADSA v8.0 uses a **fixed dollar risk** per trade, not a fixed lot size. This is the correct approach for:
- Deriv synthetics (variable contract specs)
- Cross-asset portfolios
- Accounts of any size

**Formula**: `Risk Per Trade ($)` input → sent as `vol_dollar` to TradeSgnl EA → EA calculates lots using contract specs server-side.

### SL and TP Placement

| Component | Default | Logic |
|-----------|---------|-------|
| SL | ATR × 1.5 | Placed beyond volatility noise |
| TP1 | ATR × 2.0 | First target — triggers BE automation |
| TP2 | ATR × 3.5 | Full target — triggers trail automation |
| TP3 (optional) | ATR × 5.0 | Extended target for strong momentum days |

### Risk:Reward Philosophy

- Every trade is structured with **minimum 1:2 R:R** (SL vs TP2)
- TP1 at 1:1.3 R:R moves you to **break-even** — from that point the trade is free
- TP2 at 1:2.3 R:R is where **real extraction happens**
- TP3 is harvesting the move — only available if trail is alive

### Position Sizing Example

```
Account: $500
Risk Per Trade: $10 (2%)
Asset: XAUUSD, current ATR(14) = 3.2 (320 pips on 5m)
SL distance: 3.2 × 1.5 = 4.8 (480 pips)
Lot size calculated by EA: $10 / (480 pips × $1/pip per 0.01 lot) = 0.02 lots
```

You never touch the lot size. The EA calculates it from the dollar risk.

---

## 9. MT5 EXECUTION VIA TRADESGNL

TradeSgnl is an MT5 EA that listens to TradingView alerts and executes trades. ADSA v8.0 is purpose-built to "zombie" this EA — meaning Pine Script controls all logic, and the EA blindly executes the instructions.

### Alert Message Format (Entry)

```
LICENSE_ID,SYMBOL,buy,vol_dollar=10,sl_price=1.08450,tp1_price=1.08650,pct1=0.33,tp2_price=1.08850,pct2=0.67,exent=1
```

| Field | Value | Meaning |
|-------|-------|---------|
| `vol_dollar` | Your risk in $ | EA calculates lots from this |
| `sl_price` | Exact SL price | Calculated as entry ± ATR×1.5 |
| `tp1_price` | First target price | 33% close here |
| `pct1` | 0.33 | 33% of position closed at TP1 |
| `tp2_price` | Full target price | 67% close here |
| `pct2` | 0.67 | Remaining 67% at TP2 |
| `exent` | 1 | Execute immediately at market |

### Alert Setup in TradingView

1. Right-click chart → **Add Alert**
2. Condition: **ADSA v8.0** → **alert() function calls only**
3. Message: `{{strategy.order.alert_message}}`
4. Webhook URL: your TradeSgnl webhook endpoint
5. Fire: **Once Per Bar Close** (prevents duplicate entries)

---

## 10. TRADE AUTOMATION — BE, TRAIL, PARTIAL CLOSE

After entry, the EA needs instructions to manage the trade. These are sent as separate modify alerts — no Python required.

### Break-Even on TP1

When price hits TP1:
```
LICENSE_ID,SYMBOL,modifybuy,sl_price=1.08505
```
SL is moved to entry + buffer (configurable in `[TradeSgnl EA]` group → `BE Buffer Points`). Trade is now risk-free.

### Trailing Stop on TP2

When price hits TP2:
```
LICENSE_ID,SYMBOL,modifybuy,trail=1,trail_start=0.0032,trail_step=0.0016
```
`trail_start` = 1.5× current ATR. `trail_step` = 0.5× ATR. The EA now manages the trail server-side — the trail tightens as price moves.

### Enabling Automation

In `[⚙️ TradeSgnl EA]` group:
- `✅ Auto BE on TP1` — move SL to BE when TP1 is hit
- `✅ Auto Trail on TP2` — activate EA trail when TP2 is hit
- `BE Buffer Points` — how many points above entry to set BE SL

---

## 11. TIMEZONE & SESSION SETUP (NAIROBI/EAT)

Nairobi runs on **East Africa Time (UTC+3)**. Here is how global sessions map to your clock:

| Session | UTC Open | EAT Open | Character |
|---------|---------|---------|-----------|
| Tokyo / Asia | 00:00 | 03:00 | Range, low liquidity |
| London Open | 07:00 | **10:00** | 🔑 First major move of the day |
| NY Open | 13:00 | **16:00** | 🔑 Highest liquidity, largest candles |
| London Close | 16:00 | 19:00 | Fakeouts common — be careful |
| NY Close | 22:00 | 01:00 | Wrap up, no new trades |

### Best Times to Trade from Nairobi

```
09:30–10:30 EAT  → Pre-London setup (look for liquidity sweeps before the open)
10:00–12:00 EAT  → London open — ADSA's prime hunting window
14:00–16:00 EAT  → London/NY overlap run-up — momentum extension plays
16:00–18:00 EAT  → NY open — second wave, especially on USD pairs
```

### Configuring Timezone in ADSA

In `[📊 Dashboard]` settings:
- `🕐 UTC Offset` → **3**
- `Timezone Label` → **EAT**

The dashboard will then show:
```
👑 LONDON SESSION (10:xx EAT)
🗽 NY SESSION (16:xx EAT)
🗼 TOKYO/ASIA (03:xx EAT)
```

The session detection logic remains on UTC (correct for all markets). Only the **display** converts to your local time.

---

## 12. DASHBOARD GUIDE

The ADSA cognitive dashboard is a live trading plan — readable in one glance.

```
══════════════════════════════════════
 ⏱️ FRACTAL 4-LAYER SYNC
 L1 Sovereign (1D): BULL
 L2 Anchor   (1H): BULL
 L3 Filter   (15): BULL
 L4 Exec    (5): BULL
 🔄 FULL ALIGNMENT ↑

 📊 CORE SIGNALS
 XAUUSD | 5  2645.50
 👑 LONDON SESSION (10:xx EAT)  |  ATR: High (3.24)
 📅 Daily Context: Price ABOVE PDH — bullish momentum
 🦞 Claw Trail: 🟢 BULL
 🎯 Confidence: 73% (need 60%)
 ATM 🟢  Regime 📈
 VWAP 📈  Fib 📈
 RSI 📈 (61)  MACD: 📈

 🦞 Claw 🟢  M5:▲  M15:▲  H1:▲

 🧠 5-LAYER AI NARRATIVE  [8.5/15]
 📈 BULLISH BIAS (Strong)
 ├─ L1 Sovereign: Daily EMA9 > EMA21 ✓
 ├─ L2 Anchor: H1 structure bullish ✓
 ├─ L3 Filter: 15m MACD expanding ✓
 ├─ L4 Exec: RSI above 50, Stoch bull ✓
 └─ Liquidity: PDH broken, swing high cleared ✓

 💡 AGENT ADVICE
 "All layers aligned. Claw confirms. Wait for pullback
  to trail before entry — confidence at 73%."
══════════════════════════════════════
```

### Dashboard Placement
Settings → `📍 Placement`:
- `Top-Right` — recommended for clean chart
- `Bottom-Left` — good on small screens
- `Bottom-Center` — for dual-monitor setups

---

## 13. BACKTESTING METHODOLOGY

### The Right Approach to Backtesting ADSA

Pine Script's Strategy Tester gives you **historical performance data** but requires careful interpretation for intraday strategies.

#### Step 1 — Baseline Test
1. Set chart to **5m, 500+ bars** (adjust `Max bars back` if needed)
2. `Commission`: match your broker (e.g. 0.0002 for Forex, 0.1% for crypto)
3. `Slippage`: set 1-2 ticks for Forex, 3-5 for synthetics
4. Run with **Conservative mode** (60% confidence threshold)
5. Record: Win rate, Profit Factor, Max Drawdown, Avg trade duration

#### Step 2 — Confluence Validation
Good signal = multiple layers lit simultaneously. When reviewing trades:
- Did the Claw agree at entry? (Check `claw_exec_trend`)
- Were all 4 MTF EMAs aligned?
- Was confidence above threshold?
- What was the session? (London/NY trades should outperform Asia)

#### Step 3 — What Good Numbers Look Like

| Metric | Minimum | Target | Excellent |
|--------|---------|--------|-----------|
| Win Rate | 45% | 55% | 65%+ |
| Profit Factor | 1.3 | 1.8 | 2.5+ |
| Max Drawdown | < 20% | < 12% | < 8% |
| Avg R:R | 1:1.5 | 1:2.2 | 1:3+ |
| Trades/week | — | 3-7 | — |

#### Step 4 — Walk-Forward Validation
- Backtest on **Jan–Sep** of a given year
- Forward-test Oct–Dec (don't touch settings)
- If performance degrades more than 30%, re-examine which layer is misfiring

#### Red Flags in Backtests
- High win rate (>75%) with low profit factor → SL too wide, TP too close
- Good PF but massive drawdown → position sizing too aggressive
- Works on one TF but fails on adjacent TF → signal is TF-dependent (normal, just document it)
- More than 50% of profits from 3 trades → overfitting to outlier moves

---

## 14. LIVE TRADING CHECKLIST

Run this checklist **before every trading session**:

### Pre-Session (before London open = before 10:00 EAT)
```
□ Check the 1D chart — what is the daily bias?
□ Mark PDH and PDL on your chart
□ Mark the most recent swing high and low
□ Is ATR in High or Low regime? (Low ATR = no trades)
□ Is the sovereign TF aligned with the direction you want to trade?
□ Check economic calendar — any high-impact news in next 2 hours?
□ TradeSgnl EA is running on MT5 with your license active
□ Alerts are set up in TradingView with webhook pointing to your EA
```

### During Session
```
□ Only take signals during London (10:00–12:00 EAT) or NY (16:00–18:00 EAT)
□ Confidence must be above your chosen threshold
□ Claw must be green (bull) or red (bear) — no trades on neutral Claw
□ All 4 MTF EMA layers aligned preferred — minimum 3 of 4
□ Never enter against the sovereign TF
□ Maximum 2 open trades simultaneously
□ If first trade hits TP1 and moves to BE — look for second trade
```

### Post-Session
```
□ Wait for daily report (fires at configured UTC hour)
□ Review: Did every trade match the signal conditions?
□ Document any missed signals (above threshold but you hesitated)
□ Note any false signals — which layer was misaligned?
```

---

## 15. PYRAMIDING PROTOCOL

Pyramiding in ADSA means **adding to a winning position** at defined points, not randomly doubling down.

### When Pyramiding is Allowed

1. First trade is at **break-even** (TP1 hit, SL moved to entry+buffer)
2. Confidence score is **still above threshold** on a fresh signal
3. Claw trail is **still in the same direction** (no flip)
4. The new entry is at **a lower risk price** (closer to the trail = smaller SL)

### Pyramid Position Size Rule

```
Pyramid lot = First lot × 0.5
```

Never pyramid at full size. The pyramid adds to your conviction at reduced risk — it is not a second full trade.

### Example: Gold (XAUUSD) Pyramid

```
1st entry: Buy 0.02 lots at 2645. SL at 2640. TP1 at 2648.
Price hits 2648 (TP1) → partial close 33% → SL moved to 2645.50 (BE+buffer)
New signal fires at 2647 with same bull alignment.
Pyramid: Buy 0.01 lots at 2647. SL at 2643. TP at 2655.
Result: Two trades running. First is free-riding. Second has 1:2 R:R.
```

### When NOT to Pyramid
- If the first trade has NOT yet hit TP1 (you are still at risk)
- If Claw has flipped direction at any timeframe
- If ATR has dropped to Low (volatility dried up)
- If you are approaching the London Close (16:00 EAT) with no NY catalyst

---

## 16. TUNING CHEAT SHEET BY MARKET

### XAUUSD (Gold)
```
Execution TF: 5m or 15m
ATR Length: 14
ATR Multiplier (SL): 1.5–2.0
TP1 mult: 2.0   TP2 mult: 3.5   TP3 mult: 5.0
Confidence mode: Moderate (45%)
Best sessions: London open (10:00 EAT), NY open (16:00 EAT)
Notes: High ATR = large SL — keep risk_per_trade conservative. Respect PDH/PDL hard.
```

### Deriv Volatility 75 Index
```
Execution TF: 1m or 5m
ATR Length: 10
ATR Multiplier (SL): 2.0 (V75 has fast rejection wicks)
TP1 mult: 1.5   TP2 mult: 3.0
Confidence mode: Moderate (45%)
Best sessions: 24/7 — but follow NY session for trending behaviour
Notes: vol_dollar is essential here — lots are non-standard. Never manually set lots on V75.
```

### EURUSD / GBPUSD
```
Execution TF: 15m
ATR Length: 14
ATR Multiplier (SL): 1.5
TP1 mult: 2.0   TP2 mult: 3.0
Confidence mode: Conservative (60%)
Best sessions: London open and NY open only — avoid overlap with economic news
Notes: Enable VWAP filter for Forex — helps reduce false signals during ranging sessions.
```

### US30 / NAS100
```
Execution TF: 5m
ATR Length: 14
ATR Multiplier (SL): 1.2 (indices move fast)
TP1 mult: 1.8   TP2 mult: 3.2
Confidence mode: Moderate (45%)
Best sessions: NY only (16:00–19:00 EAT)
Notes: Check futures gap at NY open. If gap > 50 pts, wait for the first 5m candle to close before trading.
```

### Deriv Crash / Boom 1000
```
Execution TF: 1m
Trade direction: Crash = SELL only. Boom = BUY only.
ATR Multiplier (SL): 3.0 (spike events need room)
TP1 mult: 1.0   TP2 mult: 2.0
Confidence mode: Aggressive (30%) — trend is synthetic, not macro-driven
Notes: Disable VWAP filter. Disable Fib Trend. Use Claw direction as primary gate.
       Never trade against the synthetic trend direction.
```

---

## 17. KNOWN BEHAVIOURS & EDGE CASES

### Bar Replay vs Live
ADSA uses `barstate.isconfirmed` to prevent signal triggers mid-bar. In live trading the signal **fires on bar close** — not in real time within the bar. This is intentional to prevent false entries.

### request.security() Repainting
The MTF Claw values (`claw_m5`, `claw_m15`, `claw_h1`) use `barstate.isconfirmed` inside `request.security()`. This eliminates repainting on live bars. Historical bars will show the confirmed close value only.

### Orphan Trade Protection
If a new signal fires while a trade is still open (e.g. the previous trade never hit SL or TP), ADSA v8.0 automatically marks the previous trade as closed in the performance array before recording the new trade. No phantom trades accumulate in the tracker.

### Regime Filter
- When `enableRegimeFilter = true`, signals only fire if the regime (RSI-based) matches the direction
- VWAP and Fib Trend are **layered independently** — they each narrow the set of valid regimes but are NOT combined into a single gate
- If VWAP is off and Fib is on: the regime check is `RSI regime AND Fib direction`
- If both are on: `RSI regime AND VWAP AND Fib direction` (all three must agree)

### Timezone and Session Logic
Session detection (for trade timing) is always UTC-based. The `vp_localHour` variable is used **only for display** in the dashboard. If you change `tz_offset_hours`, it will NOT change which hours the agent trades — it only changes what clock time you see on the dashboard.

### Strategy Tester Limitations
- Pine Script Strategy Tester cannot simulate partial closes perfectly — TP1/TP2 show as separate exit orders
- The `pct1/pct2` fields in TradeSgnl alerts handle actual partial close on MT5 — the Strategy Tester is for directional validation only, not exact P&L simulation

---

## QUICK REFERENCE CARD

```
┌─────────────────────────────────────────────┐
│  ADSA v8.0 — TRADE DECISION TREE            │
│                                             │
│  1. Is ATR HIGH?          No → Skip         │
│  2. Is Claw aligned?      No → Skip         │
│  3. Is score ≥ 6?         No → Wait         │
│  4. Is confidence ≥ 60%?  No → Skip         │
│  5. Is session active?    No → Skip         │
│     (London or NY only)                     │
│  6. ENTER — alert fires to MT5              │
│                                             │
│  After TP1 → SL to break-even (auto)        │
│  After TP2 → Trail activated (auto)         │
│  After TP3 → Manual decision or hold        │
└─────────────────────────────────────────────┘
```

---

## VERSION HISTORY

| Version | Key Changes |
|---------|------------|
| v7.0 | Original SUPREME AGENT — core ATM + VWAP + regime engine |
| v8.0 | + Claw Protocol integration (ratcheting ATR trail MTF stack) |
| v8.0 | + Confidence Engine (weighted scoring, Conservative/Moderate/Aggressive) |
| v8.0 | + Smart RSI extreme signals (75+/25- momentum entries) |
| v8.0 | + TradeSgnl BE/Trail automation (no Python required) |
| v8.0 | + Orphan trade protection (dpt[] array always accurate) |
| v8.0 | + vol_dollar fixed for Deriv synthetics |
| v8.0 | + regimeBullish/Bearish CE10156 fix (sequential ternaries) |
| v8.0 | + Nairobi/EAT timezone support (configurable UTC offset + label) |

---

*Built with Claude (Anthropic) — Liquidity Extraction Protocol for Intraday Traders*
