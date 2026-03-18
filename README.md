# Omori Law — Nifty 50 Trading Strategy

> **Disclaimer:** This is a research-based analytical framework, not financial advice. Always use proper risk management and never risk capital you cannot afford to lose.

---

## What is Omori Law?

Omori's Law comes from seismology. After a major earthquake, aftershocks decay in frequency following a power law:

$$n(t) = \frac{K}{(t + c)^p}$$

| Symbol | Name | Meaning in Markets |
|--------|------|--------------------|
| `n(t)` | Aftershock rate | Volatility level at time `t` after crash |
| `K` | Productivity | Severity/energy of the crash |
| `c` | Time offset | Early-phase smoothing constant |
| `p` | Decay exponent | How fast volatility normalizes |
| `t` | Time since crash | Trading bars elapsed since mainshock |

The idea: a market crash is the **mainshock**, and subsequent volatility spikes are **aftershocks** that decay predictably over time.

---

## Nifty 50 — Indicator Parameters

These are the calibrated values for NSE:NIFTY on the **Daily (1D)** timeframe.

### Crash Detection

| Parameter | Value | Reason |
|-----------|-------|--------|
| Mainshock Threshold | `12%` | Nifty has routine 8–10% corrections; 12%+ signals a true crash |
| Lookback for Recent High | `60 bars` | ~3 months of daily bars to define the "peak" |

### Omori Law Parameters

| Parameter | Value | Reason |
|-----------|-------|--------|
| K – Productivity | `1.2` | Moderate scaling; Nifty crashes are severe but not S&P-level liquid |
| c – Time Offset | `2.0` | Wider early smoothing due to circuit breakers and auction-open gaps |
| p – Decay Exponent | `0.70` | **Key parameter.** Emerging market memory effect — aftershocks linger longer than developed markets |

### Volatility / Aftershock Settings

| Parameter | Value | Reason |
|-----------|-------|--------|
| ATR Period | `20` | Standard 1-month rolling ATR |
| Aftershock Spike Multiplier | `1.8` | 1.8× average ATR = confirmed spike; filters out noise |

### p-value Guide by Crash Type

| Crash Type | Recommended `p` | Example |
|------------|-----------------|---------|
| Global contagion | `0.60 – 0.75` | COVID Mar 2020, GFC 2008 |
| Domestic shock | `0.80 – 0.90` | Election result, RBI surprise |
| Flash crash / single-day spike | `1.10 – 1.30` | Budget day panic |
| Sustained bear market | `0.50 – 0.65` | 2008 full bear, 2015–16 slowdown |

> **Tuning rule:** If the red histogram spikes outlast the orange decay curve → lower `p`. If the curve stays elevated after spikes die → raise `p`.

---

## The Four Seismic Zones

The indicator divides post-crash time into four zones based on the decay value:

```mermaid
flowchart TD
    CRASH["⚡ Mainshock Detected\nDrawdown ≥ 12% from 60-bar high"]
    CRASH --> ACTIVE

    ACTIVE["🔴 Active Zone\ndecay > 0.6\nBars 1–15 approx"]
    ACTIVE -->|"decay drops below 0.6"| ELEVATED

    ELEVATED["🟠 Elevated Zone\ndecay 0.3 – 0.6\nBars 15–40 approx"]
    ELEVATED -->|"decay drops below 0.3"| FADING

    FADING["🟡 Fading Zone\ndecay 0.1 – 0.3\nBars 40–80 approx"]
    FADING -->|"decay drops below 0.1"| CALM

    CALM["🟢 Calm Zone\ndecay < 0.1\nNormal market resumes"]

    style CRASH fill:#E24B4A,color:#fff,stroke:#A32D2D
    style ACTIVE fill:#FCEBEB,color:#791F1F,stroke:#E24B4A
    style ELEVATED fill:#FAEEDA,color:#633806,stroke:#EF9F27
    style FADING fill:#EAF3DE,color:#27500A,stroke:#639922
    style CALM fill:#E1F5EE,color:#085041,stroke:#1D9E75
```

---

## Trading Strategy by Zone

```mermaid
flowchart LR
    subgraph RED ["🔴 Active Zone  |  decay > 0.6"]
        direction TB
        R1["Strategy: Avoid / Hedge"]
        R2["Position size: 0–10%"]
        R3["Action: Stay cash\nSell covered calls\nBuy put options"]
        R4["Stop loss: Tight 2–3%"]
        R5["Instruments: Put options\nLiquid / debt funds"]
        R1 --> R2 --> R3 --> R4 --> R5
    end

    subgraph ORANGE ["🟠 Elevated Zone  |  decay 0.3–0.6"]
        direction TB
        O1["Strategy: Cautious buy"]
        O2["Position size: 20–35%"]
        O3["Action: Buy strong sectors\nEnter on aftershock dips\nWait for spike then buy"]
        O4["Stop loss: Medium 4–5%"]
        O5["Instruments: Large caps\nITM call options"]
        O1 --> O2 --> O3 --> O4 --> O5
    end

    subgraph YELLOW ["🟡 Fading Zone  |  decay 0.1–0.3"]
        direction TB
        Y1["Strategy: Scale in"]
        Y2["Position size: 50–70%"]
        Y3["Action: Add to winners\nEnter Nifty ETF\nSIP aggressively"]
        Y4["Stop loss: Wide 6–8%"]
        Y5["Instruments: Nifty ETF\nQuality mid caps"]
        Y1 --> Y2 --> Y3 --> Y4 --> Y5
    end

    subgraph GREEN ["🟢 Calm Zone  |  decay < 0.1"]
        direction TB
        G1["Strategy: Full position"]
        G2["Position size: 80–100%"]
        G3["Action: Ride the trend\nNifty futures long\nBuy call options"]
        G4["Stop loss: Trail 8–10%"]
        G5["Instruments: Futures\nMomentum stocks\nCall options"]
        G1 --> G2 --> G3 --> G4 --> G5
    end

    RED --> ORANGE --> YELLOW --> GREEN
```

---

## Decision Flow — What to Do Right Now

```mermaid
flowchart TD
    START(["Open Nifty chart\nCheck Omori indicator"])
    START --> Q1{"Has a crash\nbeen detected?"}

    Q1 -->|"No — pre-crash"| NORMAL["Normal market mode\nUse standard tools\nRSI, MA, etc."]
    Q1 -->|"Yes"| Q2{"What zone\nis active?"}

    Q2 -->|"🔴 Active"| A1["PROTECT capital\nDo NOT buy dips\nConsider puts"]
    Q2 -->|"🟠 Elevated"| A2{"Are FIIs\nnet sellers?"}
    Q2 -->|"🟡 Fading"| A3["BEST entry zone\nScale in steadily\nFocus on quality"]
    Q2 -->|"🟢 Calm"| A4["Full position\nTrend follow\nRaise stop trails"]

    A2 -->|"Yes"| A2Y["Stay defensive\nAct as if still Active\nMonitor weekly FII data"]
    A2 -->|"No"| A2N["Begin selective entries\nBuy on histogram spikes\nthat undercut decay curve"]

    A1 --> CHECK["Check: Are red histogram\nbars undercutting\nthe orange curve?"]
    A2N --> CHECK
    A3 --> CHECK
    A4 --> CHECK

    CHECK -->|"Yes — spikes fading"| UPGRADE["Zone improving\nConsider upgrading\nstrategy tier"]
    CHECK -->|"No — spikes above curve"| HOLD["Stay in current zone\nDo not rush entries"]

    UPGRADE --> START
    HOLD --> START

    style A1 fill:#FCEBEB,color:#791F1F,stroke:#E24B4A
    style A2 fill:#FAEEDA,color:#633806,stroke:#EF9F27
    style A2Y fill:#FAEEDA,color:#633806,stroke:#EF9F27
    style A2N fill:#FAEEDA,color:#633806,stroke:#EF9F27
    style A3 fill:#EAF3DE,color:#27500A,stroke:#639922
    style A4 fill:#E1F5EE,color:#085041,stroke:#1D9E75
```

---

## The Golden Rule

> **Never fight the aftershocks.**
>
> The red histogram (actual volatility) tells the truth. The orange Omori decay curve tells you the *expected* rate of decay.
>
> - If red spikes **outlast** the orange curve → the market has more structural damage than the model predicted. Stay defensive longer. Lower your `p`.
> - If red spikes **undercut** the orange curve → the market is healing faster than expected. This is your green light to start scaling in.
> - The **gap between actual and predicted** is the signal. Not the price. Not the news.

---

## Nifty-Specific Adjustments

### FII Flow Override
The Omori model captures **volatility decay** but not **capital flow dynamics**. When transitioning from Elevated → Fading zone, always cross-check:

- If FIIs are **net sellers** week-on-week → treat the zone as one level more defensive
- If FIIs are **net buyers** → trust the Omori signal; accelerate entries

### Circuit Breaker Gap
Nifty's 10%/15% circuit breakers mean the first day of a crash is often distorted. The `c = 2.0` parameter handles this — it smooths the first 2 bars after the mainshock so the decay curve doesn't overfit to the halt noise.

---

## Historical Test Cases

| Crash | Date | Drawdown | Suggested p | Zone Duration |
|-------|------|----------|-------------|---------------|
| COVID crash | Feb–Mar 2020 | ~38% | `0.65` | Active: ~15 bars |
| GFC | Jan 2008–Mar 2009 | ~60% | `0.60` | Multiple mainshocks |
| China spillover | Aug–Sep 2015 | ~16% | `0.80` | Active: ~8 bars |

> **Test tip:** In TradingView, use the date range bar to jump to these periods. Observe whether the orange decay curve envelope matches the rhythm of the red histogram spikes. Tune `p` until they align.

---

## Quick Reference Card

```
INDICATOR: Omori Law (omori_law_market.pine)
CHART:     NSE:NIFTY — Daily (1D)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PARAMETER              VALUE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Mainshock Threshold    12%
Lookback High          60 bars
K – Productivity       1.2
c – Time Offset        2.0
p – Decay Exponent     0.70  ← tune this
ATR Period             20
Aftershock Multiplier  1.8
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ZONE      DECAY     SIZE    ACTION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔴 Active  > 0.6    0–10%   Protect / hedge
🟠 Elevated 0.3–0.6 20–35%  Cautious entries
🟡 Fading  0.1–0.3  50–70%  Scale in (best zone)
🟢 Calm    < 0.1   80–100%  Full deployment
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

*Strategy developed using Omori's Law (1894) applied to financial markets. Cross-reference with FII flow data, sector rotation, and macro context before executing trades.*
