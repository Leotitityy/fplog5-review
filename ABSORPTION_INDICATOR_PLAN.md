# FPLOG5 Absorption Indicator — Design Plan

## Mission
Build a **NQ-first TradingView Pine Script v6 absorption indicator** using native footprint data. It must be useful for discretionary trading and research, but remain understandable and fast. Do not turn it into a giant multi-factor black box.

TradingView now exposes native `request.footprint()` data in Pine v6, including total buy/sell volume, delta, row-level volume/delta, POC, VAH/VAL and row imbalances. This is the primary data source. Do NOT recreate footprint data with lower-timeframe hacks. citeturn0search0turn0search2turn0search3

## Core definition
Absorption is not simply "big volume" or "large delta".

### Bullish absorption
Aggressive sellers are active at/near the low, but price fails to continue down. Evidence:
1. Sell pressure is concentrated in the lower part of the footprint.
2. The lower rows have meaningful sell volume / negative delta.
3. The bar rejects the low: close is materially above the low relative to the bar range.
4. Preferably the bar is testing an important location: prior low, session low, VWAP deviation, VAL, or a recent swing.

### Bearish absorption
Mirror image:
1. Buy pressure concentrated in the upper part of the footprint.
2. Upper rows have meaningful buy volume / positive delta.
3. Price rejects the high.
4. Preferably the bar is testing an important location: prior high, session high, VWAP deviation, VAH, or recent swing.

## Signal architecture
Use a **0–100 score**, not dozens of binary conditions.

Suggested components:

- **Footprint pressure: 0–30**
  - extreme sell/buy delta relative to recent bars
  - concentration of aggressive volume near the relevant extreme
- **Rejection / failed auction: 0–25**
  - wick/body relationship
  - close location within the bar
  - excursion beyond a recent extreme followed by recovery
- **Location: 0–20**
  - session high/low
  - previous day high/low
  - VWAP and configurable VWAP standard-deviation bands
  - footprint VAH/VAL/POC
- **Imbalance evidence: 0–15**
  - sell imbalance clusters near lows for bullish absorption
  - buy imbalance clusters near highs for bearish absorption
- **Relative volume: 0–10**
  - current footprint volume vs rolling baseline

Do not require every component. The score should degrade gracefully when one feature is unavailable.

## Footprint implementation
Use one native `request.footprint()` request. TradingView documents that footprint rows are accessible through `footprint.rows()`, sorted from lowest to highest price, and individual rows expose buy volume, sell volume, total volume, delta and imbalance state. citeturn0search3

Inputs should stay limited to the useful ones:
- ticks per row: default suitable for NQ, but configurable
- value-area %: default 70
- imbalance %: default around 300
- lookback for relative statistics
- signal threshold

For NQ, start with **1 tick per row** and validate row count/data availability before changing it. Do not hard-code assumptions that only work on NQ.

Scan the footprint rows once per bar and calculate only the statistics actually needed:
- total buy/sell volume
- total delta
- POC / VAH / VAL
- lower-zone sell concentration
- upper-zone buy concentration
- max relevant row delta
- number/strength of relevant imbalance rows
- position of POC inside the candle

Avoid expensive drawing or large arrays unless they materially improve the signal.

## Context / location layer
Use simple, high-value context:
- session VWAP
- 1st and 2nd VWAP standard-deviation bands
- previous day high/low
- current session high/low
- recent swing high/low
- footprint VAH/VAL/POC

TradingView's VWAP implementation supports standard-deviation bands around anchored VWAP values; use the same concept for location rather than inventing a complicated volatility model. citeturn0search13

Do not make every level mandatory. Location should increase confidence, not manufacture a signal.

## Signal types
Only three visual states in the first version:

1. **Strong Bullish Absorption** — score >= strong threshold
2. **Bullish Absorption** — score >= normal threshold
3. **Bearish equivalents**

Plot a compact marker at/near the bar extreme. The marker should communicate direction and strength without covering the chart.

Optional debug mode may show:
- score
- delta
- relative volume
- relevant imbalance count
- location reason

Debug mode OFF by default.

## Anti-noise rules
Implement these before adding more features:
- minimum relative volume requirement
- minimum footprint row count / data availability check
- minimum rejection requirement
- cooldown between same-direction signals
- do not fire when the footprint object is `na`
- no lookahead
- no future-bar information
- signal should be based only on data known on the current completed bar

The indicator must explicitly distinguish **"no footprint data"** from **"no absorption"**.

## Reversal vs continuation
Do not assume every absorption means an immediate reversal.

Store two concepts internally:
- `absorption_score`: quality of the trapped/aggressive activity
- `location_score`: quality of the location

A high absorption score at a bad location may indicate continuation/temporary absorption rather than a tradeable reversal. This distinction is important for later research.

## Research outputs
Add an optional research table, not dozens of labels. It should show the latest bar's:
- Bull score
- Bear score
- delta / delta %
- relative volume
- rejection score
- location score
- imbalance evidence
- POC position
- VA width
- data availability

Later, the logger can export these features for Python analysis. Do not optimize thresholds from visual examples alone.

## What NOT to build yet
Do not add:
- machine learning
- dozens of moving averages
- automatic entries/exits
- stop-loss/TP logic
- complicated market-regime classification
- multi-timeframe request jungle
- CVD as a mandatory signal
- 20+ configurable thresholds
- fancy footprint rendering

Those can be research branches after the base signal has evidence.

## Validation plan
Test in this order:
1. Compile correctness in Pine v6.
2. Confirm footprint data exists on NQ and row counts are sensible.
3. Visually inspect obvious absorption examples and obvious non-absorption examples.
4. Compare score distributions across at least several weeks of NQ data.
5. Measure forward excursion after signals: MFE/MAE at fixed horizons.
6. Separate signals by location: VWAP bands, PDH/PDL, session extremes, VAH/VAL, neutral area.
7. Only then tune thresholds.

Use TradingView's Pine Profiler if runtime becomes an issue; TradingView recommends profiling before optimizing bottlenecks. citeturn0search6

## Critical research principle
The objective is **not maximum number of signals** and not maximum historical win rate. The objective is to find a feature combination that identifies genuine failed auctions / absorbed aggression and remains useful out-of-sample.

A signal that occurs 3 times a day with clean context is more valuable than a noisy signal that appears on every candle.

---

# CLAUDE — EXECUTION INSTRUCTIONS

You are responsible for implementing the first working version in the **`Leotitityy/fplog5-pine`** repository.

## Required workflow
1. Read this plan completely.
2. Inspect `src/fplog5.pine`. The current file is only a placeholder, so build the first real implementation there. fileciteturn7file0
3. Implement **Pine Script v6**.
4. Use native `request.footprint()` as the core order-flow source.
5. Keep the code modular: inputs → footprint extraction → feature calculations → scoring → signals → plots/table.
6. Do not add unrelated features.
7. Keep all calculations readable enough that another engineer can audit the logic.
8. Add comments explaining WHY a feature is used, not comments that merely restate the code.
9. Commit the implementation to the Pine repo.

## Important implementation constraints
- Use only one unique `request.footprint()` call.
- Handle `na` footprint data safely.
- Avoid repainting / lookahead.
- Do not use future bars.
- Do not silently substitute ordinary volume when footprint data is missing.
- Do not create a signal solely from delta magnitude.
- Normalize relative features so thresholds are not tied to one absolute NQ volume number.
- Keep default settings conservative.

## First version acceptance criteria
The first implementation is accepted only if:
- it compiles as Pine v6;
- it uses native footprint data;
- it produces separate bullish/bearish absorption scores;
- it uses row-level footprint information, not just bar delta;
- it includes rejection + location + pressure rather than a single trigger;
- it has normal/strong signal thresholds;
- it has a small debug/research table;
- it is visually clean;
- no obvious repaint/lookahead logic exists.

## After implementation
Create/update a short `IMPLEMENTATION_NOTES.md` in the review repo containing:
- exact features implemented;
- default parameters;
- any TradingView/Pine limitations encountered;
- known weaknesses;
- 3–5 concrete ideas for the next research iteration.

Do NOT tune the system until the first implementation is complete and reviewable.
