# splunk-internal-phishing-behavioral-detection

Behavioral Internal Phishing Detection in Splunk Overview

This project demonstrates the design and implementation of a behavioral anomaly detection framework in Splunk to identify internal-to-internal phishing activity conducted through compromised accounts.

Traditional email security controls primarily focus on external threats. When an adversary compromises a legitimate internal account, malicious emails can originate from trusted identities and bypass perimeter-based detection mechanisms. This project addresses that detection gap using statistical/behavioral baselining and risk-based alert aggregation within Splunk.

Problem Statement

Internal phishing campaigns leveraging valid accounts are difficult to detect because:

-  Email originates from legitimate users

-  No obvious malicious domain indicators

-  Traditional IOC-based detections may not trigger

-  Activity blends into normal communication patterns

The objective was to design a behavioral detection model capable of identifying abnormal internal email patterns while maintaining operational feasibility and minimizing false positives.

Dataset

-  350 synthetic internal email events

-  Baseline normal internal communication activity

-  3 injected simulated phishing attack windows

-  Fields normalized to Splunk Common Information Model (CIM)

Detection Architecture

  1️⃣ Behavioral Baselining

  -  Hourly binning by sender

  -  Sender-specific statistical baseline

  Threshold defined as:

    Threshold = average hourly volume + (2 × standard deviation)

  This approach allows dynamic modeling of normal behavior rather than static rule thresholds.

  2️⃣ Behavioral Enrichment Conditions

  In addition to volume spike detection, two enrichment indicators were incorporated:

  -  ≥ 4 unique recipients within one hour

  -  ≥ 3 new sender-recipient relationships within one hour

  These conditions improve detection fidelity beyond simple volume-based alerts.

  3️⃣ Risk Scoring Model

  Weighted additive scoring:

  | Behavioral Indicator | Condition                         | Weight |
  | -------------------- | --------------------------------- | ------ |
  | Volume spike         | Hourly count > baseline threshold | +30    |
  | Unique recipients    | ≥ 4                               | +20    |
  | New relationships    | ≥ 3                               | +25    |

  Severity Levels:

  -  Low: 0–29

  -  Medium: 30–49

  -  High: 50+

  This structure aligns with risk-based alerting principles and reduces alert fatigue by prioritizing composite behavioral anomalies.

Validation Results

-  3 simulated phishing attack windows

-  2 successfully detected

-  1 low-volume scenario remained below statistical threshold

Overall detection rate: 66.67% (Minimal false positives during baseline activity)

The missed detection highlights the tradeoff between sensitivity and alert stability inherent in statistical anomaly detection models. The decision was made not to overtune thresholds to preserve operational feasibility.

Artifacts

-  SPL correlation search

-  Risk scoring model

-  Investigation dashboard (Internal Email Anomaly View)

Example Detection Logic (SPL):

    index=internal_email sourcetype=email_logs
    | bin _time span=1h
    | stats count AS hourly_count dc(recipient) AS unique_recipients BY sender _time
    | eventstats avg(hourly_count) AS avg_count stdev(hourly_count) AS stdev_count BY sender
    | eval threshold = avg_count + (2 * stdev_count)
    | eval risk_score = 0
    | eval risk_score = risk_score + if(hourly_count > threshold, 30, 0)
    | eval risk_score = risk_score + if(unique_recipients >= 4, 20, 0)
    | eval severity = case(risk_score >= 50, "High",
                      risk_score >= 30, "Medium",
                      risk_score < 30, "Low")
    | where risk_score > 0


Key Takeaways

-  Statistical baselining is effective for detecting behavioral anomalies

-  Composite risk scoring improves prioritization

-  Detection sensitivity must be balanced against false positive rates

-  Behavioral analytics can be implemented within existing SIEM infrastructure

-  Detection engineering requires data normalization and validation before rule development

Future Enhancements

-  Z-score normalization

-  Time-of-day baselining

-  Role-based sender modeling

-  Adaptive thresholding

-  SOAR integration for automated containment

-  Expansion to include link analysis and attachment metadata

Skills Demonstrated

-  Splunk SPL development

-  Behavioral anomaly modeling

-  Statistical thresholding

-  Risk-based alerting

-  Detection engineering workflow

-  Data normalization (CIM alignment)

-  Alert validation & performance evaluation

Intended Audience

This project is relevant for:

-  SOC teams seeking behavioral detection strategies

-  Detection engineers building anomaly-based models

-  Security engineers implementing risk-based alerting in SIEM platforms
