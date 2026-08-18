# GPT REVIEW — V2 REQUIRED FIXES

Claude V1 was reviewed by GPT. Do NOT optimize thresholds or add new features yet. First fix the structural issues below and compile-test in TradingView.

## Required fixes

1. **Exclude current bar from session high/low location tests.**
   - The current bar must not qualify as "near session high/low" merely because it created that high/low.
   - Compare against the prior session state where appropriate.

2. **Exclude current bar from swing high/low location tests.**
   - `ta.highest()` / `ta.lowest()` including the current bar creates a self-referential location signal.
   - Use prior bars for the reference level.

3. **Make imbalance evidence directional and location-aware.**
   - Bullish absorption: prioritize **sell imbalance near the lower footprint / low**.
   - Bearish absorption: prioritize **buy imbalance near the upper footprint / high**.
   - Do not use the single longest imbalance chain anywhere in the footprint as generic evidence.

4. **Fix relative-volume baseline.**
   - Do not feed missing/invalid footprint bars into the baseline as zero volume.
   - Baseline statistics must use only bars with valid footprint data.

5. **Do not optimize score thresholds yet.**
   - Keep the current thresholds only as temporary defaults.
   - No parameter fitting until score distributions and MFE/MAE have been collected.

6. **Keep exactly ONE `request.footprint()` call.**

7. **Add research/debug fields for:**
   - footprint row count
   - lower-zone sell concentration
   - upper-zone buy concentration
   - relevant sell imbalance count
   - relevant buy imbalance count
   - POC position
   - location score
   - total bull score
   - total bear score

8. **Verify Pine API names against TradingView's official Pine v6 documentation.**
   - In particular `footprint.rows()` and volume-row imbalance methods.
   - Do not rely on third-party documentation when an official API reference is available.

9. **Compile-test in TradingView.**
   - Do not claim the script works until it compiles.
   - If an API detail cannot be verified, document the exact uncertainty.

10. **Do not add complexity just for complexity's sake.**
   - The goal is a robust NQ absorption detector, not a giant scoring framework.
   - Prefer a small number of meaningful features over dozens of correlated ones.

## Important conceptual rule

A large delta/imbalance is NOT absorption by itself.

Absorption should require some combination of:
- aggressive volume/imbalance,
- price reaching an extreme or meaningful location,
- failure to continue,
- rejection/reclaim.

The indicator should detect the **failure of aggressive participants to move price**, not simply unusually high volume.

## After V2

Only after the structural fixes are complete:
1. collect score distributions;
2. inspect bull/bear symmetry;
3. measure forward MFE/MAE at fixed horizons;
4. determine whether each score component adds information;
5. only then consider threshold calibration.

Do not optimize to a small historical sample.

— GPT review, 2026-08-18