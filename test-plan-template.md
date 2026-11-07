# A/B Test Plan Template

> This document is a reusable planning template. The example values below are based on the synthetic checkout experiment in this project; replace them before using the plan for a different experiment.

## 1. Experiment Overview

| Field | Planned value |
|---|---|
| Experiment name | Checkout page conversion test (illustrative) |
| Owner | To be assigned |
| Product / area | E-commerce checkout |
| Planned start date | To be decided |
| Planned end date | To be decided |
| Status | Planning / Not launched |
| Experiment type | A/B test, two-sided comparison |

## 2. Objective and Hypothesis

**Objective:** Evaluate whether the redesigned checkout page changes the visitor-to-purchase conversion rate.

- **Control:** Existing checkout page.
- **Variant:** Redesigned checkout page.
- **Primary metric:** Conversion rate (conversions divided by eligible assigned users).

**Null hypothesis (H₀):** The control and variant conversion rates are equal.

**Alternative hypothesis (H₁):** The control and variant conversion rates differ.

\[
H_0: p_C = p_V
\qquad
H_1: p_C \ne p_V
\]

## 3. Metric Definition

| Item | Definition |
|---|---|
| Primary metric | Conversion rate |
| Numerator | Number of eligible users who complete the defined purchase action |
| Denominator | Number of eligible users assigned to the experiment |
| Attribution window | Define before launch |
| Unit of analysis | Unique eligible user |
| Guardrail metrics | Define product-specific metrics before launch |

The conversion event, eligibility rules, attribution window, and unit of analysis must be fixed before the experiment begins.

## 4. Statistical Design

| Parameter | Planned value |
|---|---:|
| Control conversion rate (baseline) | 10% |
| Variant conversion rate (design alternative) | 12% |
| Absolute MDE | 2 percentage points |
| Relative lift at the design alternative | 20% |
| Significance level (α) | 0.05 |
| Target power (1 − β) | 0.80 |
| Test | Two-sided, two-proportion test |
| Traffic allocation | 50% control / 50% variant |
| Required sample per group | 3,839 |
| Required total sample | 7,678 |

**MDE interpretation:** The 2-percentage-point MDE is the effect this design is powered to detect at the selected alpha and power. It is not automatically the smallest effect that is commercially worthwhile.

**Sample-size method:** Two-proportion normal approximation. Recalculate the sample size if the baseline rate, MDE, alpha, power, or allocation ratio changes.

## 5. Traffic and Duration

| Item | Planned value |
|---|---:|
| Total eligible traffic | Approximately 5,000 users/day |
| Traffic split | 50% / 50% |
| Minimum calendar duration | 7 complete days |
| Sample-size target | 7,678 total users |

**Stopping requirement:** Meet both conditions before the final analysis:

1. The experiment has run for at least 7 complete days.
2. The pre-specified total sample size has been reached.

At the assumed traffic level, the sample-size calculation alone implies approximately 1.54 days. The 7-day minimum is an operational choice to cover a full weekly cycle.

Possible traffic plans:

- Allocate approximately 22% of eligible traffic to the experiment to collect roughly 7,678 users over 7 days.
- Alternatively, use all eligible traffic for at least 7 days and accept a larger-than-required sample and greater exposure to the variant.

Choose the traffic plan before launch. Do not stop or change allocation in response to interim p-values.

## 6. Randomization and Data Quality

Complete these items before launch:

- [ ] Confirm users are assigned randomly.
- [ ] Confirm assignment is persistent for each user.
- [ ] Confirm users cannot enter both groups.
- [ ] Validate the 50/50 allocation and monitor for sample-ratio mismatch.
- [ ] Define bot, employee, test-account, and duplicate-event handling.
- [ ] Validate that the conversion event is recorded consistently in both groups.
- [ ] Confirm the data pipeline and experiment logging.

These are planning checks; they are not all implemented in the synthetic simulation in this repository.

## 7. Monitoring and Stopping Rule

**Primary analysis:** Run one final two-sided hypothesis test after both the minimum duration and planned sample size conditions are met.

**Decision threshold:** Declare statistical significance only if the final p-value is below 0.05, subject to data-quality checks and guardrail metrics.

- [ ] Do not stop early because a nominal p-value falls below 0.05.
- [ ] Do not repeatedly test the result using an unadjusted 0.05 threshold.
- [ ] If interim decisions are required, specify a sequential-testing or alpha-spending method before launch.
- [ ] Document any operational pause or data-quality incident.

## 8. Analysis and Reporting

Report at minimum:

- [ ] Users assigned and analyzed in each group.
- [ ] Conversions and conversion rate in each group.
- [ ] Absolute difference in conversion rates.
- [ ] Relative lift.
- [ ] Test statistic and p-value.
- [ ] Confidence interval for the treatment effect.
- [ ] Sample-ratio and data-quality checks.
- [ ] Guardrail metric results.
- [ ] Final decision and limitations.

A statistically significant result does not by itself establish that the change is practically or commercially valuable.

## 9. Decision Framework

| Result | Action |
|---|---|
| Significant positive result and guardrails acceptable | Consider rollout, subject to product review |
| Not statistically significant | Do not claim a win; assess uncertainty and whether a new test is justified |
| Significant negative result | Do not roll out; investigate potential harm |
| Data-quality or randomization issue | Treat the result as unreliable and investigate before deciding |

## 10. Sign-off

| Role | Name | Approval / date |
|---|---|---|
| Experiment owner |  |  |
| Product owner |  |  |
| Data / analytics reviewer |  |  |
| Engineering reviewer |  |  |

---

## Project Simulation Reference

The companion notebook uses synthetic data to demonstrate the design and operating characteristics. Its recorded simulation results are:

| Scenario | Simulations | Result |
|---|---:|---:|
| Null: 10% vs 10% | 10,000 | 4.78% false-alarm rate |
| Alternative: 10% vs 12% | 10,000 | 80.75% estimated power |
| Null with 8 unadjusted interim checks | 5,000 | 17.68% false-alarm rate |

These are simulation results under the stated assumptions, not results from a live product experiment.
