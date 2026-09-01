# A/B Test Analysis — Surge Pricing Cap Experiment

A portfolio project simulating a full experimentation workflow for a ride-hailing
platform: hypothesis → power calculation → randomization → analysis →
guardrails → recommendation.

> All data is **synthetically generated** (`data/generate_experiment_data.py`).
> No real company or user data is used.

## The experiment

**Hypothesis:** Capping peak-hour surge pricing at 1.5x (instead of letting it
run uncapped up to ~2.5x) will reduce price-driven cancellations and raise
completed-ride volume — at the cost of some per-ride revenue, and possibly at
the cost of captain earnings/willingness to accept peak-hour rides.

- **Control (A):** uncapped surge pricing
- **Treatment (B):** surge capped at 1.5x
- **Unit of randomization:** customer (to avoid interference effects within a city)
- **Duration:** 14 days, peak hours only (8-10am, 6-9pm)
- **Primary metric:** ride completion rate
- **Secondary metric:** average revenue per completed ride
- **Guardrail metric:** captain-cancellation rate (proxy for captain dissatisfaction)

## What's inside

| File | Purpose |
|---|---|
| `data/generate_experiment_data.py` | Simulates randomized assignment + ride outcomes with a realistic, non-trivial effect baked in |
| `analysis/ab_test_analysis.py` | Full analysis: sample size/power calc, SRM check, significance tests, guardrail check, recommendation |

## Analysis workflow

1. **Sample size / power** — back-calculates the required sample size to detect a 2pp lift in completion rate at 80% power, and confirms the experiment was adequately powered
2. **Randomization check (SRM)** — a chi-square test confirms the control/treatment split isn't badly skewed, plus a per-city balance check
3. **Primary metric** — two-proportion z-test on completion rate, with Wilson confidence intervals
4. **Secondary metric** — Welch's t-test on revenue per completed ride (unequal variances assumed)
5. **Guardrail metric** — chi-square test on captain-cancellation rate
6. **Recommendation** — synthesizes all three results into a ship/hold/iterate call, rather than looking at completion rate in isolation

## Result (on the generated dataset)

The capped-surge treatment showed a **statistically significant lift in
completion rate** (+2.5pp, p<0.001) but **also a significant drop in
revenue per ride** and a **significant rise in captain-side cancellations**
— a realistic "it's not a clean win" outcome. The write-up in
`ab_test_analysis.py`'s output frames the actual trade-off a BA would have to
present to stakeholders, rather than a one-line "ship it."

## Running it

```bash
cd data && python generate_experiment_data.py
cd ../analysis
pip install numpy pandas scipy statsmodels
python ab_test_analysis.py
```

## Skills demonstrated

- Experiment design: randomization unit choice, sample size/power calculation
- Inferential statistics: two-proportion z-test, Welch's t-test, chi-square test, confidence intervals
- Guardrail-metric thinking (not optimizing one metric in isolation)
- Translating statistical output into a business recommendation

## Suggested resume bullets

- Designed and analyzed a simulated A/B test on a surge-pricing cap for a ride-hailing platform (~59K ride requests, 12K randomized customers), running a power analysis, SRM check, and two-proportion z-test to validate a +2.5pp completion-rate lift (p<0.001)
- Identified a guardrail trade-off (captain-cancellation rate rose significantly under treatment) using a chi-square test, and translated the combined results into a ship/iterate recommendation rather than a single-metric read
