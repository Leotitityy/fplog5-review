# GPT REVIEW — V3 FROM FIRST 3-MIN TEST

The first 3-minute NQ chart test looks **promising but too noisy**.

The important observation is NOT that the indicator is bad. Several markers appear to catch meaningful reversals/turning points. The problem is that too many lower-quality markers fire around the same move.

## Visual diagnosis

### What is working
- There are clear examples where a marker appears near a local extreme and price subsequently reverses strongly.
- The directional logic is broadly behaving sensibly: bullish markers cluster around downside exhaustion and bearish markers around upside rejection.
- The indicator is already producing useful candidates on 3m, so do not redesign the whole architecture.

### Main problem
**Signal precision is too low.**

The current scoring allows a bar to accumulate points from several weak/correlated conditions without proving that actual absorption occurred.

## V3 objective
Do NOT simply raise the global threshold. That would hide the problem rather than solve it.

We need a **quality gate** before a score becomes a signal.

### Bullish absorption should require
At least:
1. Valid footprint data.
2. Evidence of aggressive selling / sell imbalance in the lower footprint area.
3. Rejection / failed continuation to the downside.
4. At least ONE meaningful location context OR a sweep/failure event.

Mirror for bearish:
1. Valid footprint.
2. Aggressive buying / buy imbalance in the upper footprint area.
3. Rejection / failed continuation upward.
4. At least ONE meaningful location context OR sweep/failure event.

## Important: do not over-filter
Do NOT require every possible condition simultaneously.

The goal is to remove the obvious weak markers while preserving the strong early reversals visible in the 3m test.

## Recommended scoring changes

### 1. Footprint evidence must matter more than generic volume
Current relative-volume score is symmetric and can add points without directional absorption evidence.

Keep relative volume as confirmation, but do not allow it to create a signal by itself.

### 2. Imbalance should be a gate/strong confirmation
A directional relevant-zone imbalance should be present for the highest-quality signals.

For the first V3 test, use:
- normal signal: relevant directional imbalance OR a very strong footprint pressure condition;
- strong signal: relevant directional imbalance REQUIRED.

### 3. Location should not be able to manufacture a signal
VWAP/VAH/VAL/session/swing proximity are contextual evidence only.
They should not compensate for weak footprint pressure + weak rejection.

### 4. Avoid correlated point stacking
PDL + session low + swing low can describe essentially the same location.
Do not treat five nearby location labels as five independent pieces of evidence.

Better concept:
- `locationContext = 0..1`
- multiple nearby location references increase confidence modestly, but saturate quickly.

### 5. Keep thresholds conservative for now
Do not optimize to this one screenshot.
Start V3 with approximately:
- normal threshold: 60–65
- strong threshold: 75–80

But the quality gates matter more than these exact numbers.

## Add research/debug fields
Show in the debug table:
- bull score
- bear score
- directional footprint evidence
- rejection score
- location context count/strength
- relevant imbalance streak
- relative volume
- whether quality gate passed
- final signal type

## Test protocol after V3
Use NQ 3m and inspect at least 2–3 different sessions/days.

Do NOT judge V3 from one screenshot.

Record approximately:
- total signals
- strong signals
- obvious winners
- obvious false positives
- signals during trends vs signals at extremes

If V3 removes 50% of markers but keeps most of the obvious good reversals, that is progress.

If it removes the good early signals too, loosen the gate instead of adding more features.

## Do NOT add
- machine learning
- more dozens of indicators
- automatic trade entries
- parameter optimizer
- complex market regime model

We are still validating the core absorption detector.

— GPT review after first live 3m test, 2026-08-18