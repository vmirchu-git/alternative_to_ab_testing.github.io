<p align="center">
  <img src="assets/regression_chart.png" alt="Pre/post release regression trend-break chart" width="100%">
</p>

<p align="right"><a href="README.ru.md">Русская версия →</a></p>

# Pre/Post Release Regression Test — an Alternative to A/B Testing

![Python](https://img.shields.io/badge/Python-pandas-14131a?style=flat-square&labelColor=14131a&color=7a1f2b)
![statsmodels](https://img.shields.io/badge/statsmodels-OLS%20regression-14131a?style=flat-square&labelColor=14131a&color=7a1f2b)
![SQL](https://img.shields.io/badge/SQL-automated%20queries-14131a?style=flat-square&labelColor=14131a&color=7a1f2b)

> **Note on source code.** This repository documents methodology, design decisions, and results only. The implementation is proprietary to the employer where this project was built and is not published here.

A statistical method and automated pipeline for measuring the true effect of a release when splitting traffic into A/B groups isn't available or practical — separating what actually changed from what was just trend or seasonality, and flagging it automatically instead of relying on someone eyeballing a graph.

## At a glance

| | |
|---|---|
| **Target accuracy** | ≤5% error vs. gold-standard A/B — achieved on the comparisons it was validated against |
| **Applicable to** | any release, from mass-market changes to single-intent tweaks |
| **Robustness check** | validated across 7 / 14 / 30-day analysis windows |
| **Status** | designed as a reusable methodology; not actively run today since the underlying intent-creation process it supported was discontinued |

## Problem

Product teams commonly compared 2 weeks before vs. 2 weeks after a release to judge its impact. Without a proper control group, this comparison is vulnerable to trend, seasonality, and external factors — a release could look like it hurt a metric that was actually already declining, or look like it helped when the metric was simply recovering on its own. Decisions were often made by eye, and negative changes could go unnoticed.

## Approach

**Core method.** For a given release date, fit two ordinary least squares regressions on the metric's time series — one on the window before the release, one on the window after:

```
pre-release:  k_before · (t - t0) + b_before
post-release: k_after  · (t - t0) + b_after
```

Two derived quantities separate what changed:

- **Δb = b_after − b_before** — the release effect: a level shift right at the moment of the release, independent of the existing trend.
- **Δk = k_after − k_before** — the trend change: whether the release also changed the *direction* the metric was already moving in, not just its level.

Statistical significance of both is assessed via the standard error of the fitted coefficients, so a shift only counts as a real effect once it clears a significance threshold — not just because two lines happen to look different.

**Automation.** SQL queries to pull the relevant metric windows are generated automatically per intent, and result descriptions are auto-drafted from the fitted coefficients — the analyst reviews and interprets rather than builds the report from scratch each time.

**Validation.** The method's accuracy was checked against releases where a real A/B test was also available, and its stability was tested by varying the analysis window (7, 14, and 30 days) to confirm the estimated effect wasn't an artifact of window choice.

**Iterative development.** Built in three stages: manual validation on 1–2 intents, then automated report generation, then consolidated visualizations and dashboards for broader use.

## Results

- Achieved the ≤5% error target relative to gold-standard A/B testing on the releases it was validated against.
- Applicable to both mass releases and narrowly scoped, single-intent changes — the same method scales down without modification.
- Surfaced degradations that limited-traffic A/B tests had missed, by using the full pre/post time series instead of a traffic-split sample.

## Business impact

- Gave product teams a way to evaluate release impact even when A/B testing wasn't feasible, instead of defaulting to an eyeballed before/after comparison.
- Reduced the risk of shipping — and not noticing — a release that quietly hurt a metric.
- Turned release analysis into an automated, repeatable report instead of a one-off manual investigation each time.

## Tech stack

Python · pandas · NumPy · statsmodels (OLS) · Seaborn · Matplotlib · SQL

---

<sub>Individual project completed as part of a Data Analytics role. Described here for portfolio purposes; production code is not publicly available.</sub>
