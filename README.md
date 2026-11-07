<h1 align="center">A/B Test Design & Sizing</h1>

<p align="center">
  <em>How many users do you need, how long should you wait, and when can you trust the result?</em>
</p>

<p align="center">

![Python](https://img.shields.io/badge/Python-3.12%2B-blue?logo=python&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?logo=numpy&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?logo=pandas&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-8CAAE6?logo=scipy&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-11557c)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)
![Status](https://img.shields.io/badge/Status-Ready%20for%20review-blue)

</p>

---

## TL;DR

A new checkout page shows a higher conversion rate. Is it a real win or just luck?

This project sets up the test properly *before* any data is collected, then proves with simulation that the setup works.

- **Sample size:** 3,839 users per group (7,678 total) to detect a lift from 10% to 12%.
- **Test quality:** In 10,000 simulated tests, the false-alarm rate was 4.78% (target: 5%) and the detection rate was 80.75% (target: 80%).
- **Biggest risk:** Checking results every day and stopping at the first "significant" result pushed the false-alarm rate from about 5% to **17.68%**.
- **Rule:** Fix the sample size and stop date in advance. Do not stop early.

---

## Results at a Glance

### Experiment Setup

| Item | Value |
|---|---:|
| Control conversion rate | 10% |
| Variant conversion rate | 12% |
| Smallest lift to detect (absolute) | 2 percentage points |
| Smallest lift to detect (relative) | 20% |
| Significance level (α) | 5% |
| Target power | 80% |
| Required users per group | 3,839 |
| Total required users | 7,678 |
| Minimum run time | 7 days |

### Simulation Results

<table>
<tr>
<td align="center" width="33%">
<h2>4.78%</h2>
<strong>False-Alarm Rate</strong>
<br/>
<sub>10,000 simulations, no real difference</sub>
<br/>
<sub>95% CI: 4.36% – 5.20%</sub>
</td>
<td align="center" width="33%">
<h2>80.75%</h2>
<strong>Power</strong>
<br/>
<sub>10,000 simulations, real 2-point lift</sub>
<br/>
<sub>95% CI: 79.98% – 81.52%</sub>
</td>
<td align="center" width="33%">
<h2>17.68%</h2>
<strong>False-Alarm Rate With Peeking</strong>
<br/>
<sub>5,000 simulations, 8 checks</sub>
<br/>
<sub>95% CI: 16.62% – 18.74%</sub>
</td>
</tr>
</table>

> [!NOTE]
> Simulation results change a little every run. The confidence intervals above quantify Monte Carlo uncertainty around the estimated rates; they are calculated from the simulation counts and are generated in the notebook using a normal-approximation interval. The 5% target lies inside the false-alarm interval, and the 80% target lies inside the power interval, which is consistent with the design targets.

---

## Table of Contents

1. [Business Problem](#business-problem)
2. [Experiment Design](#experiment-design)
3. [Sample Size and Duration](#sample-size-and-duration)
4. [Synthetic Data](#synthetic-data)
5. [Hypothesis Testing and Simulation](#hypothesis-testing-and-simulation)
6. [The Peeking Problem](#the-peeking-problem)
7. [Key Findings](#key-findings)
8. [Recommendations](#recommendations)
9. [Tech Stack](#tech-stack)
10. [Project Structure](#project-structure)
11. [How to Run](#how-to-run)
12. [Limitations](#limitations)
13. [Conclusion](#conclusion)
14. [Author](#author)

---

## Business Problem

An e-commerce company wants to test a redesigned checkout page.

| Role | Description |
|---|---|
| **Control** | Current checkout page |
| **Variant** | New checkout page |
| **Main metric** | Conversion rate |

The business hopes conversion goes from 10% to 12%. But a higher number in the variant group does not prove the new page is better. Random chance alone can create gaps.

Before launching, the team needs answers to five questions:

- How many users do we need?
- How long should the test run?
- Is the observed difference real or random?
- Will this test catch a real lift when one exists?
- What goes wrong if we check results early and stop?

---

## Experiment Design

### Parameters

| Parameter | Value |
|---|---:|
| Control conversion rate | 0.10 |
| Variant conversion rate | 0.12 |
| Significance level (α) | 0.05 |
| Power (1 − β) | 0.80 |
| Minimum detectable effect (MDE) | 0.02 |
| Daily eligible traffic | 5,000 users |
| Test type | Two-sided, two-proportion test |
| Traffic split | 50% / 50% |

### Hypotheses

- **Null (H₀):** Both pages convert at the same rate.
- **Alternative (H₁):** The two pages convert at different rates.

$$H_0: p_c = p_v \qquad\qquad H_1: p_c \neq p_v$$

### Minimum Detectable Effect (MDE)

The MDE is the smallest effect this experiment is designed to detect with the selected significance level (5%) and target power (80%). Whether a smaller effect is worth detecting is a separate business decision.

$$MDE = 0.12 - 0.10 = 0.02$$

- **Absolute lift:** 2 percentage points
- **Relative lift:** 20%

---

## Sample Size and Duration

Sample size uses the standard two-proportion formula (normal approximation) with a 5% significance level, 80% power, and a 10% baseline rate.

### Sensitivity: Smaller Lifts Need Many More Users

| Absolute MDE | Users per group | Total users | Days of traffic needed |
|---|---:|---:|---:|
| 1 percentage point | 14,749 | 29,498 | 5.90 |
| **2 percentage points (selected)** | **3,839** | **7,678** | **1.54** |
| 3 percentage points | 1,772 | 3,544 | 0.71 |

Cutting the target lift in half roughly quadruples the sample needed. That is the main trade-off in test design.

### Selected Design

$$N_{total} = 3{,}839 \times 2 = 7{,}678$$

$$Duration = \frac{7{,}678}{5{,}000} \approx 1.54 \text{ days}$$

### Why Run 7 Days and Not 1.54

Traffic alone says 1.54 days is enough. But user behavior changes between weekdays and weekends. A test that covers only part of the week can give a biased answer.

**Decision:** Run for at least 7 complete days **and** ensure the planned sample size has been reached before making the final decision.

> [!IMPORTANT]
> At 5,000 users per day, 7 days brings in about 35,000 users, far more than the 7,678 required. Two ways to handle this:
> - **Limit the test to about 22% of daily traffic** (7,678 ÷ 35,000), so the test reaches its sample size at day 7.
> - **Use all traffic for 7 days.** The test will then have more than 80% power, and can detect smaller lifts, but more users see an untested page.
>
> Decide this before launch. Do not decide it after seeing results.

---

## Synthetic Data

The data is generated with random yes/no outcomes (Bernoulli trials) using known conversion rates. Because the true answer is known, we can check whether the testing method finds it.

| Group | Users | True conversion rate |
|---|---:|---:|
| Control | 3,839 | 10% |
| Variant | 3,839 | 12% |
| **Total** | **7,678** | — |

Output file: `data/simulated_ab_test_run.csv`

Any single run will show rates a bit different from 10% and 12%. That is normal random variation, and it is the exact reason we run tests like this.

---

## Hypothesis Testing and Simulation

One test result tells you very little about how good the method is. So the test is repeated thousands of times, and we count how often it is right or wrong.

### Scenario A: No Real Difference

Both groups convert at 10%.

| Metric | Result |
|---|---:|
| Simulations | 10,000 |
| Significance threshold | 0.05 |
| False-alarm rate (Type I error) | **4.78%** |

This is how often the test wrongly says "there is a difference" when there is none. It should be close to 5%, and it is.

### Scenario B: Real 2-Point Lift

Control converts at 10%, variant at 12%.

| Metric | Result |
|---|---:|
| Simulations | 10,000 |
| True difference | 2 percentage points |
| Power | **80.75%** |

This is how often the test correctly finds the real lift. It should be close to 80%, and it is.

### Reading the P-value Charts

- **No real difference:** p-values spread out evenly between 0 and 1.
- **Real difference:** p-values bunch up near 0.

---

## The Peeking Problem

### What Is Peeking?

Peeking means checking results again and again during the test and stopping the first time the p-value drops below 0.05.

Each check is another chance to be fooled by luck. The more checks, the higher the chance of a false win.

### Simulation Setup

| Parameter | Value |
|---|---:|
| Control conversion rate | 10% |
| Variant conversion rate | 10% (no real difference) |
| Simulations | 5,000 |
| Checks per test | 8 |
| Threshold per check | 0.05 |
| False-alarm rate | **17.68%** |

### Comparison

| Approach | False-alarm rate |
|---|---:|
| Check once at the planned end | 4.78% |
| Check 8 times and stop at the first "win" | 17.68% |

Under the no-effect scenario, eight unadjusted interim checks increased the experiment-level false-alarm rate from 4.78% to 17.68% (about 3.7 times higher). This means that, when there is truly no difference, approximately 17.68% of experiments using this peeking procedure would incorrectly declare a statistically significant result. It does **not** mean that 17.68% of all observed winning tests are false wins.

> [!WARNING]
> If the team must check results early, use a method built for it, such as sequential testing or alpha spending. These adjust the threshold at each check so the overall false-alarm rate stays at 5%.

---

## Key Findings

1. **Smaller lifts need far more users.** Halving the lift roughly quadruples the sample size.
2. **The test is only as good as its power.** At 80% power, 1 in 5 real wins will still be missed. This is a choice, not a flaw.
3. **The method holds up.** Simulated false-alarm rate (4.78%) and power (80.75%) both match the design targets.
4. **Calendar time matters as much as sample size.** A full weekly cycle protects against day-of-week bias.
5. **Peeking is costly.** Eight unadjusted checks increased the false-alarm rate by more than threefold (3.7×).
6. **Simulation is a useful check.** With known ground truth, you can test the method itself, not just the data.

---

## Recommendations

If this were a real test, I would:

- **Write the plan before launch:** metric, lift, sample size, run time, and stop rule, all fixed in advance (see `test-plan-template.md`).
- **Meet both stopping conditions:** run for at least 7 complete days and reach the pre-specified sample size. With approximately 22% traffic allocation, the target sample is reached around day 7. With all traffic, the sample will be larger than required after 7 days.
- **Check the traffic split first.** If a 50/50 split comes out as 52/48, something is broken in assignment and the result cannot be trusted.
- **Report the size of the lift with a confidence interval,** not just "significant or not."
- **Block early looks** unless a sequential method is in place.

---

## Tech Stack

| Tool | Use |
|---|---|
| Python | Core programming |
| NumPy | Random data and simulation |
| Pandas | Data handling |
| SciPy | Statistical tests |
| Matplotlib | Charts |
| Jupyter Notebook | Interactive analysis |

---

## Project Structure

```text
ab-test-design-sizing/
│
├── data/
│   └── simulated_ab_test_run.csv   # Simulated experiment data
│
├── ab_test_sizing.ipynb            # Main analysis notebook
├── test-plan-template.md           # Reusable A/B test plan template
├── requirements.txt                # Python dependencies
├── README.md
└── .gitignore
```

---

## How to Run

### Requirements

- Python 3.12 or newer
- Git
- Jupyter Notebook, or VS Code with the Jupyter extension

### Steps

**1. Get the project files**

```bash
git clone https://github.com/satyam-katara/ab-test-design-sizing.git
cd ab-test-design-sizing
```

If you already have the project folder locally, open that folder and continue with the next step.

**2. Create a virtual environment**

```bash
python -m venv .venv
```

**3. Activate it**

| System | Command |
|---|---|
| Windows (PowerShell) | `.\.venv\Scripts\Activate.ps1` |
| macOS / Linux | `source .venv/bin/activate` |

**4. Install dependencies**

```bash
pip install -r requirements.txt
```

**5. Start Jupyter and open the notebook**

```bash
jupyter notebook
```

Open `ab_test_sizing.ipynb` and run the cells from top to bottom.

---

## Limitations

- **Simulation project, not a live experiment.** The checkout example and traffic assumptions are illustrative; no real users or production checkout page were tested.
- **Synthetic data.** Real users are messier than random coin flips.
- **Approximate formula.** Sample size uses a normal approximation, which is less accurate for very low conversion rates or small samples.
- **Simulation noise.** Estimated rates shift slightly every run. The confidence intervals shown above quantify Monte Carlo uncertainty in those estimated rates; they are not confidence intervals for the treatment effect in a real experiment.
- **Missing real-world issues.** Seasonality, sample ratio mismatch, and users who appear more than once are not modeled.
- **Seven days is a rule of thumb.** It does not remove every possible bias, such as holidays or campaigns running during the test.

---

## Conclusion

A good A/B test is decided before the data arrives. This project covers the full path: sizing the test, generating data, running the hypothesis test, checking it with 10,000 simulations, and showing how peeking breaks it.

The main lesson: comparing two conversion rates is not enough. You also need the right sample size, a fixed stopping rule, and a test whose error rates you have actually measured.

---

## Author

**Satyam Katara** — Final-year B.Tech CSE (Data Science) student at Hindustan College of Science and Technology (HCST), Mathura.

- GitHub: [satyam-katara](https://github.com/satyam-katara)
- LinkedIn: [satyamkatara](https://www.linkedin.com/in/satyamkatara/)
- Portfolio: [satyam-katara-portfolio.vercel.app](https://satyam-katara-portfolio.vercel.app)

---

<p align="center">
  <strong>A/B Test Design & Sizing</strong><br/>
  <em>Plan the test first. Trust the result second.</em>
</p>
