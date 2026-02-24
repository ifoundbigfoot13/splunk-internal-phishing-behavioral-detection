# Behavioral Internal Phishing Detection in Splunk

## Overview

This project demonstrates the design and validation of a behavioral anomaly detection framework in Splunk to identify internal-to-internal phishing activity originating from compromised accounts.

Traditional email security tools primarily focus on blocking external threats. When an attacker compromises a legitimate internal account, malicious emails can originate from trusted identities and bypass perimeter-based detection mechanisms. This project addresses that detection gap using statistical baselining, behavioral enrichment, and risk-based alert aggregation within Splunk.

---

## Problem Statement

Internal phishing campaigns leveraging valid credentials are difficult to detect because:

- Email originates from legitimate users
- No obvious malicious domains or external indicators
- Activity blends into normal communication patterns
- Static IOC-based detections may not trigger

The objective was to build a behavioral detection model capable of identifying anomalous internal communication while maintaining operational stability and minimizing false positives.

---

## Detection Architecture

### 1. Hourly Behavioral Baselining

- Sender-specific hourly aggregation
- Statistical threshold defined as:


threshold = average(hourly_count) + (2 × standard deviation)


This dynamic approach adapts to each sender’s normal communication pattern rather than relying on fixed volume thresholds.

---

### 2. Behavioral Enrichment Conditions

In addition to statistical spike detection, two enrichment signals are incorporated:

- **Unique recipients ≥ 4** within one hour  
- **New sender-to-recipient relationships ≥ 3** within one hour  

New relationships are identified by comparing hourly communication pairs against a baseline lookup derived from normal activity.

See `/docs/lookup_setup.md` for full lookup configuration details.

---

### 3. Risk-Based Alerting Model

Weighted additive scoring:

| Indicator | Condition | Weight |
|------------|-----------|--------|
| Volume spike | Hourly count > threshold | +30 |
| Unique recipients | ≥ 4 | +20 |
| New relationships | ≥ 3 | +25 |

Severity tiers:

- Low: 0–29  
- Medium: 30–49  
- High: 50+  

Alerts are generated only when cumulative risk exceeds defined thresholds to reduce alert fatigue.

---

## Example Detection Snippet

```spl
index=internal_email sourcetype=internal_email_live
| bin _time span=1h
| stats count AS hourly_count dc(recipient) AS unique_recipients BY sender _time
| eventstats avg(hourly_count) AS avg_count stdev(hourly_count) AS stdev_count BY sender
| eval threshold = avg_count + (2 * stdev_count)
```
The complete detection logic, including lookup-based relationship modeling and risk scoring, is available in:

`/detections/internal_phishing_behavioral_detection.spl`


---

## Validation Results

The detection model was evaluated using a synthetic internal email dataset containing controlled phishing simulations.

- Total events: 350  
- Simulated attack windows: 3  
- Successfully detected: 2  
- Detection rate: **66.67%**  
- Minimal false positives during baseline activity  

One low-volume attack did not exceed the statistical threshold. The model was intentionally not overtuned to preserve operational stability and prevent alert instability.

Full validation details are available in:

`/docs/validation_results.md`

---

## Investigation Dashboard

An investigation dashboard was developed to support SOC triage and contextual review of behavioral alerts. The dashboard provides:

- Hourly sender volume vs threshold
- Unique recipient counts
- New relationship counts
- Risk score and severity
- Recipient distribution analysis

See `/docs/dashboard_overview.md` for dashboard structure and workflow.

---

## Repository Structure


detections/
internal_phishing_behavioral_detection.spl

docs/
lookup_setup.md
dashboard_overview.md
validation_results.md

lookups/
sender_recipient_baseline.csv

images/
dashboard.png


---

## Key Takeaways

- Statistical baselining improves detection of compromised internal accounts
- Composite risk scoring reduces single-condition alert noise
- Behavioral modeling complements traditional email filtering
- Detection tuning requires balancing sensitivity with operational stability
- Validation is essential before deployment into production environments

---

## Skills Demonstrated

- Splunk SPL development
- Statistical anomaly modeling
- Lookup-driven relationship analysis
- Risk-based alerting
- Detection validation methodology
- SOC workflow design
- SIEM-based behavioral analytics

---

## Future Enhancements

- Time-of-day behavioral modeling
- Z-score normalization
- Role-based sender segmentation
- Adaptive thresholding
- SOAR integration for automated containment
