# 🦞 THE CLAW PROTOCOL — Intraday Liquidity Extraction (CLAW-X)

> **Strategy build of the Claw Protocol Liquidity Suite, wired to the TradeSgnl MT5 execution engine.**
> Pine Script is the Nervous System (senses, decides, defines risk boundaries).
> TradeSgnl is the Execution Engine (sizes lots, places orders, manages the trail on MT5).

**File:** `Intraday Liquidity Claw - Strategy.txt` (Pine Script v6, `strategy()`)

---

## 1. Architecture

```
TRIGGER     UT Bot / Smart RSI ............... the "when"
GATE        Liquidity Trail direction ........ the "which way" (absolute truth)
CONFLUENCE  MTF + Structure + RSI + VWAP + Fib + VP + 💧Liquidity Sweeps
CONFIDENCE  Weighted score ≥ Claw Mode threshold
EXECUTION   strategy.entry/exit → alert_message JSON → webhook → TradeSgnl → MT5
```

**Core philosophy:** compound small wins, aggressively reduce losses. No home runs —
high-probability extraction. SL is the Liquidity Trail (absolute price). The EA drags
SL to breakeven at +1R via `trtrig`/`trdist`. Pyramid adds are capped at 2–3, ever.

### What's new vs. the indicator version

| Subsystem | What it does |
|---|---|
| **Strategy engine** | Real `strategy.entry`/`exit`/`close` — Strategy Tester now validates direction, R-multiples, pyramid behavior |
| **Liquidity Pool Engine** | PDH/PDL, PWH/PWL, frozen Asia/London/NY session ranges, EQH/EQL clusters |
| **Sweep Detection** | Wick through a pool + close back inside = liquidity EXTRACTED. SSL sweep → long fuel, BSL sweep → short fuel (confidence bonus, never a trigger) |
| **Pyramid Engine** | Smart RSI re-ignition adds, reduced risk, hard cap (input max = 3), cooldown, SL ratchet |
| **TradeSgnl Transmitter** | JSON payloads in every order's `alert_message`, fully editable templates |
| **Circuit Breaker** | Stop opening new positions after N losing trades per day |
| **Kill-Zone Gate** | Optional: new entries only inside London/NY kill zones (exits always allowed) |
| **Dashboard telemetry** | 3 new rows: pyramid count + Σ risk, live position E/SL/TP, sweep + KZ status |

---

## 2. TradeSgnl EA Configuration (MT5 side) — set EXACTLY

| EA Setting | Value |
|---|---|
| Volume Parameter Source | **Signal Inputs** |
| TP/SL Parameter Source | **Signal Inputs** |
| SL/TP Mode | **Absolute Price Level** ← the Claw sends exact prices, not pips |
| Pending Order Entry | **Absolute Price Level** |
| Volume Type | **Percentage of Equity (Loss)** ← sizes lots from SL distance |
| Pyramid Management | **Signal** ← the Claw dictates scale-ins |
| Trailing Stop Management | **Signal** ← the Claw dictates `trtrig`/`trdist` |
| Price Distance Unit | **Currency Points** ← required for indices/CFDs/crypto |

The EA does ALL lot math (balance, leverage, tick size). The Pine never calculates lots.

---

## 3. TradingView Setup

1. Add **CLAW-X** to an intraday chart (M5 recommended; M1–M15 supported).
2. Settings → **group 13** → set your **TradeSgnl License ID**.
3. Settings → **group 12** → set risk:
   - `Risk per Strike` = **1.5** if EA Volume Type is *Percentage of Equity (Loss)*
     (1.5% of a $1,000 account ≈ the $10–15 doctrine), or hard dollars if you run a
     currency-loss volume type (then untick *Risk Value is % of Equity*).
4. Create **ONE alert**:
   - Condition: `CLAW-X` → **Order fills and alert() function calls**
   - Message: `{{strategy.order.alert_message}}`
   - Webhook URL: your TradeSgnl endpoint
5. Done. Every strike, scale-in, hard exit, SL/TP/trail close carries its own payload.

---

## 4. Payload Reference

Defaults (group 13, all editable — placeholders substituted at fire time):

```json
// STRIKE (primary entry, long)
{"license":"...","ticker":"...","action":"buy","vol":1.5,"sl_price":...,"tp_price":...,"trtrig":...,"trdist":...,"comment":"CLAW-L"}

// SCALE-IN (same comment → EA pyramids the SAME position, vol is reduced)
{"license":"...","ticker":"...","action":"buy","vol":0.75,"sl_price":...,"tp_price":...,"trtrig":...,"trdist":...,"comment":"CLAW-L"}

// HARD EXIT / SL / TP / TRAIL close
{"license":"...","ticker":"...","action":"closebuy","comment":"CLAW-L"}
```

Placeholders: `{{license}} {{ticker}} {{risk}} {{sl}} {{tp}} {{trtrig}} {{trdist}} {{entry}} {{scale_n}} {{conf}}`

- `sl_price` / `tp_price` — **absolute prices** (Liquidity Trail ± buffer / R:R target)
- `trtrig` / `trdist` — **currency points** (`price distance ÷ (mintick × Ticks-per-Point input)`).
  With defaults (`trtrig` = 1R, `trdist` = 1× trigger): when price moves +1R the EA's
  trail activates and the SL lands exactly at entry → breakeven lock, then ratchets.
- `tp_price` sends `0` when *Use Hard TP* is off (trail-only extraction).

> ⚠️ **Verify key names once:** docs.tradesgnl.com blocks automated readers, so these
> defaults follow the integration brief plus the parameter dialect already proven in
> ADSA v8.0. Run TradeSgnl's **Syntax Generator**, compare, and if any key differs,
> fix it in **Settings → group 13** — no code changes needed. CSV fallback dialect:
> `LICENSE,{{ticker}},buy,vol={{risk}},sl_price={{sl}},tp_price={{tp}},trtrig={{trtrig}},trdist={{trdist}}`

---

## 5. The Compounding Math (mechanics, not promises)

Per setup with defaults on a $1,000 account, EA volume = 1.5% equity-loss:

```
STRIKE      risk $15.00   SL = Liquidity Trail   TP = 1.5R   trail arms at +1R
ADD #1      risk  $7.50   (0.5× factor)          SL ratchets to the newer trail
ADD #2      risk  $7.50   hard cap reached → further adds are MANUAL
            ──────────
Σ nominal   $30.00 — but the trail trigger has usually already dragged the
            position SL to ≥ breakeven before adds stack, so live risk at any
            moment is typically far below the nominal sum.
```

- Losses are pre-defined and small (trail + buffer + daily circuit breaker).
- Wins bank at 1.5R or run on the EA trail after breakeven.
- The `vol` percentage compounds automatically: 1.5% of a growing equity is a
  growing dollar claw with identical risk geometry.

---

## 6. Strategy Tester Notes

- The Tester mirrors the EA: risk-based qty (`risk ÷ SL distance`), absolute SL,
  hard TP, trailing after +1R, pyramiding ≤ 3 adds.
- Use it for **directional validation and R-multiple distribution** — not exact MT5
  P&L (no swap/commission/slippage modeling; EA point values differ per broker).
- `process_orders_on_close=true`: fills happen at the close of the signal bar —
  exactly when the live alert fires. No intrabar fantasy fills.

## 7. Hard Rules (encoded, not suggested)

1. Max **2–3** automated scale-ins (`Max Auto Scale-Ins` input is capped at 3).
2. The Liquidity Trail is directional **truth** — no entry against it, hard exit on flip.
3. Pools and sweeps are **context**, never triggers. Triggers are UT Bot / Smart RSI only.
4. Daily loss circuit breaker halts NEW entries; exits are never blocked.
5. The EA owns lot sizing. The Pine owns intent and absolute risk boundaries.

## 8. Roadmap

- **Phase 1 (this build):** Pine senses → JSON → TradeSgnl executes.
- **Phase 2:** Python senses (faster, more assets) → JSON → TradeSgnl executes.
- **Phase 3:** Python + Claude senses & reasons → direct API execution (Bybit, Deriv, MT5).

---

*Educational/research tooling. Markets carry risk; nothing here is financial advice or a performance guarantee.*
