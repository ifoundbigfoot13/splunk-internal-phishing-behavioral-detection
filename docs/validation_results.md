# Detection Validation Results

## Objective

The detection model was evaluated against a synthetic internal email dataset to measure its ability to identify internal-to-internal phishing behavior conducted via compromised accounts.

The goal of validation was not to achieve 100% detection, but to balance anomaly sensitivity with operational stability and reduced false positives.

---

## Dataset Overview

- Total events: 350 internal email logs
- Dataset type: Synthetic internal enterprise communication
- Normal baseline activity: Majority of dataset
- Simulated attack windows: 3 controlled phishing scenarios
- Fields normalized to Splunk CIM

---

## Evaluation Method

The model was validated using the following criteria:

1. Statistical spike detection (avg + 2σ per sender)
2. Unique recipient volume ≥ 4 within 1 hour
3. New sender-to-recipient relationships ≥ 3 within 1 hour
4. Composite weighted risk scoring model

Alerts were generated only when cumulative risk exceeded defined thresholds.

---

## Results

- Simulated attack windows: 3
- Successfully detected: 2
- Missed: 1 (low-volume attack below statistical spike threshold)
- Overall detection rate: **66.67%**

Baseline (normal) activity generated minimal false positives under defined thresholds.

---

## Missed Detection Analysis

The undetected scenario involved a lower-volume phishing attempt that did not exceed the statistical baseline threshold.

This was an intentional design tradeoff:

- Lowering thresholds would increase detection sensitivity.
- However, this would likely introduce alert instability and increase false positives.
- The model was tuned to prioritize operational feasibility over aggressive sensitivity.

This aligns with real-world detection engineering practices where stability and signal quality must be balanced against theoretical maximum detection coverage.

---

## Operational Considerations

Key design decisions included:

- Avoiding overtuning to achieve artificial 100% detection.
- Maintaining sender-specific statistical baselines.
- Using composite risk scoring to reduce single-condition alerts.
- Supporting SOC triage via contextual dashboard visualization.

The detection logic demonstrated consistent behavior across controlled test cases and maintained alert relevance.

---

## Performance Summary

| Metric | Result |
|--------|--------|
| Total Events | 350 |
| Attack Windows | 3 |
| True Positives | 2 |
| False Positives | Minimal |
| Detection Rate | 66.67% |

---

## Conclusion

The behavioral detection framework successfully identified high-risk internal phishing behavior in the majority of simulated attack scenarios while maintaining operational stability.

The validation process demonstrates:

- Statistical anomaly modeling
- Behavioral enrichment integration
- Risk-based alert aggregation
- Tradeoff awareness in detection tuning

The model provides a practical foundation for expanding behavioral detection capabilities within a SIEM environment.
