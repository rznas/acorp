# Alerts and anomaly detection

Deep reference for alert configuration and the anomaly-detection detectors summarized in SKILL.md. Alerts are supported on **trends only**.

## Table of contents
- [Alert types and thresholds](#alert-types-and-thresholds)
- [Check frequency and timing](#check-frequency-and-timing)
- [Notification destinations](#notification-destinations)
- [Breakdowns](#breakdowns)
- [Anomaly detection overview](#anomaly-detection-overview)
- [Detector catalog](#detector-catalog)
- [Configuration parameters](#configuration-parameters)
- [Simulating before saving](#simulating-before-saving)
- [Tuning tips](#tuning-tips)
- [Auto-disabled alerts](#auto-disabled-alerts)

## Alert types and thresholds

- **has value** — absolute check against a number.
- **increases by / decreases by** — relative check on change; enables **percentage** thresholds (% change between two values).
- Set at least one bound: **more than**, **less than**, or both.
- **absolute value** thresholds work on any trend value; **percentage** thresholds only on relative alerts.

## Check frequency and timing

Check intervals: every 15 minutes (requires Boost, Scale, or Enterprise), hourly, daily, weekly, monthly.

- **Check ongoing period** — also evaluate the current incomplete period (this week/month) so you're alerted as soon as it crosses, not only after the period closes.
- **Skip weekends** — for daily alerts that shouldn't fire on weekends.
- **Quiet hours** — define up to 5 blocked windows (HH:MM, 24h, project timezone, each ≥30 min). Checks scheduled during quiet hours run when the window ends. Preset: overnight 10 PM–7 AM.

## Notification destinations

Subscribed users always get **in-app notifications** (clicking opens the insight). Add inline: **email**, **Slack** (connect integration, pick channel), **Discord** (webhook URL), or generic **webhook** URL. Multiple destinations per alert are allowed.

## Breakdowns

On a trend with a breakdown, the alert fires when **any** breakdown value breaches the threshold.

## Anomaly detection overview

Beta — enable under Settings > Feature previews -> **Anomaly Detection Alerts**. Instead of a fixed threshold, a statistical/ML detector learns "normal" and flags outliers. Use when you don't know what threshold to set, want subtle changes caught, or have seasonal patterns that break static thresholds.

Create: open a trend -> Actions -> Alerts -> New alert -> alert type **Anomaly detection** -> choose a detector -> set sensitivity & window -> **Simulate** on history -> set notifications -> Create alert.

## Detector catalog

13 algorithms. Z-score or MAD are the recommended starting points.

### Statistical
| Detector | Best for | How it works |
| --- | --- | --- |
| **Z-score** | General purpose | Flags points many standard deviations from the rolling mean; uses first-order differencing by default for cyclical data. |
| **MAD** (Median Absolute Deviation) | Data with outliers | Like Z-score but uses median; more robust to existing outliers; differencing on by default. |
| **IQR** (Interquartile Range) | Skewed distributions | Flags values outside the interquartile range; less sensitive to distribution shape. |

### Machine learning (PyOD library)
| Detector | Best for | How it works |
| --- | --- | --- |
| **Isolation Forest** | General, high-dimensional | Randomly partitions data; anomalies isolate with fewer partitions. |
| **KNN** | Cluster-based patterns | Flags points far from nearest neighbors. |
| **LOF** | Variable-density data | Compares local density to neighbors; good when "normal" varies by range. |
| **ECOD** | Fast, no tuning | Empirical CDFs; very fast, no hyperparameters beyond threshold/window. |
| **COPOD** | Multivariate | Copula-based; fast, parameter-light. |
| **HBOS** | Fast, large datasets | Histogram density; very fast, assumes feature independence. |
| **PCA** | Correlated metrics | Projects to lower dimensions; flags high reconstruction error. |
| **OCSVM** | Complex boundaries | Learns a boundary around normal data; good for irregular shapes. |

### Ensemble
Combines two or more detectors:
- **AND** — fires only when all sub-detectors agree (fewer false positives).
- **OR** — fires when any flags (catches more, noisier).

## Configuration parameters

| Parameter | Description | Default |
| --- | --- | --- |
| **Sensitivity** (threshold) | Anomaly probability cutoff 0–1; higher = fewer alerts. | 0.9 |
| **Window** | Historical data points used for training. | by interval (below) |

Default window by check interval:
| Interval | Window | Covers |
| --- | --- | --- |
| Hourly | 168 | 7 days |
| Daily | 90 | ~3 months |
| Weekly | 26 | ~6 months |
| Monthly | 12 | 1 year |

**Preprocessing:** **Differencing** (`diffs_n`) converts values to consecutive changes — on by default for Z-score/MAD to stop daily/weekly cycles being flagged. **Lagging** (`lags_n`) adds lagged values as features for more temporal context (useful for Isolation Forest/KNN).

## Simulating before saving

The Simulate section (below detector config) runs the detector on historical data and shows: a chart of values + anomaly scores, red dots on flagged points, and stats (total points, anomalies found, anomaly rate). Ensemble detectors show a score line per sub-detector. Change the date range dropdown to test different windows.

## Tuning tips

- Start with **Z-score** or **MAD**.
- Always **simulate** before saving.
- Set sensitivity to **0.95** for noisy/high-variance metrics (default 0.9 may over-alert).
- Use **ensemble AND** to reduce false positives.
- Keep **differencing** on for cyclical data.

## Auto-disabled alerts

PostHog auto-disables alerts with invalid config and emails subscribed users the reason. Common causes: insight changed type (trend -> funnel), the alerted series was removed, a relative alert set on a non-time-series trend (pie/number), an absolute alert given a percentage threshold, missing/invalid check frequency, or a threshold alert with no bounds. Disabled alerts do **not** self-recover — fix the config, then manually re-enable from the insight's **Alerts** tab.
