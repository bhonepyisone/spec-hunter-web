# Grading Formula Reference — Spec Hunter Web

Source: SPEC.md section 5, user decision (Bhone, June 2026)

## Battery Thresholds

| Grade | Battery % | Condition |
|-------|-----------|-----------|
| A+ | ≥85% | Excellent, rare on 3yr+ stock |
| A | ≥75% | Good, common on 1-2yr stock |
| B | ≥60% | Fair, normal wear on 3-4yr |
| C | ≥40% | Degraded but functional |
| D | <40% | Needs battery replacement |

## SSD Health Thresholds

| Grade | SSD Health % |
|-------|-------------|
| A+ | ≥95% |
| A | ≥80% |
| B | ≥60% |
| C | ≥40% |
| D | <40% |

## Body Condition Scoring

| Rating | Score |
|--------|-------|
| excellent | 100 |
| good | 75 |
| fair | 50 |
| damaged | 0 |

Body average = mean of all 9 parts (top_cover, bottom_cover, keyboard, palmrest, screen, left_side, right_side, front, back)

## Grade Calculation

```
if battery≥85 AND ssd≥95 AND bodyAvg≥90 AND no tests failed → A+
else if battery≥75 AND ssd≥80 AND bodyAvg≥70 → A
else if battery≥60 AND ssd≥60 AND bodyAvg≥50 → B
else if battery≥40 AND ssd≥40 → C
else → D
```

Test failure blocks only A+. Lower grades still achievable with test failures.

## Pricing Multipliers

| Grade | Multiplier |
|-------|-----------|
| A+ | 1.45x |
| A | 1.35x |
| B | 1.20x |
| C | 1.05x |
| D | 0.85x |

Formula: `sell_price = buy_price × multiplier`
