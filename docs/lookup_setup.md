# Lookup Setup: sender_recipient_baseline

## Purpose

The `sender_recipient_baseline` lookup is used to identify new sender-to-recipient relationships during hourly behavioral analysis.

This lookup represents known (baseline) communication pairs derived from normal internal email activity. During detection execution, any sender-recipient pair not found in this lookup is treated as a new relationship and contributes to risk scoring.

---

## Lookup File Structure

The CSV lookup must contain the following headers:


sender,recipient


Each row should represent a known baseline communication pair.

Example:


| sender | recipient |

| alice@corp.com | bob@corp.com |

| alice@corp.com | carol@corp.com |

| bob@corp.com | dave@corp.com |


---

## Step 1: Generate Baseline Relationships

Run the following search in Splunk using baseline-only (normal) activity:

```spl
index=internal_email sourcetype=internal_email_live
| stats count by sender recipient
| table sender recipient
```

⚠️ Important:
Generate this lookup using only baseline time ranges (exclude known attack windows).
If the lookup includes attack activity, new relationships will not be detected.

Export the results as CSV and save the file as:

sender_recipient_baseline.csv
