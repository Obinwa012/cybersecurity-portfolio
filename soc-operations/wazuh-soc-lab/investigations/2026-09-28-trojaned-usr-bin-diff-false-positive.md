# Investigation: Wazuh flags `/usr/bin/diff` as trojaned (false positive)

**Date investigated:** 2026-09-28
**Framework:** [Alert Investigation Framework](../docs/alert-investigation-framework.md) — first live run
**Verdict:** ✅ False positive

## The alert

Wazuh rule **510** (rootcheck — host-based anomaly detection) fired a **level 7** alert:

> Trojaned version of file `/usr/bin/diff` detected. Signature used:
> `'bash|/bin/sh|file.h|proc.h|dev/[^n]|/bin/.*sh'` (Generic).

| Field | Value |
|---|---|
| Agent | `nexus-soc-01` |
| File | `/usr/bin/diff` |
| Decoder | rootcheck |
| Rule | 510 — Host-based anomaly detection event (rootcheck) |
| Level | 7 |

A trojaned system binary is a scary alert — if real, anything executing `diff`
runs attacker code. That's exactly why it went through the full framework instead
of a gut call.

![Wazuh security events dashboard](../screenshots/01-wazuh-security-events-dashboard.jpg)
*Wazuh dashboard (last 24h) at investigation time: 4,764 total events, 4,472
authentication failures, zero level-12+ alerts — a noisy environment where triage
discipline matters.*

![Rootcheck alert detail](../screenshots/02-rootcheck-alert-detail.jpg)
*Raw alert detail: rule 510, level 7, trojaned `/usr/bin/diff`, agent `nexus-soc-01`.*

## Step 1 — Categorize ✅

`/usr/bin/diff` is a core system binary (diffutils). Only root or the package
manager should ever modify it. If it were truly infected, the impact would be
high — every invocation executes malicious code. That raised the bar for
verification: no shortcuts on a high-stakes verdict.

## Step 2 — Verify ✅

All three verification checks ran:

1. **Local package logs** — no recent updates or installs touching `diff`. Nothing
   in the logs explained a modification.
2. **Local package integrity check** — ran an integrity check against the
   known-good package to look for tampering. The file came back **clean**.
3. **External threat intel** — VirusTotal: **0/63 detections**. Community notes
   confirmed this is a **known false positive** caused by an overly-broad Wazuh
   rootcheck regex signature ([wazuh/wazuh#19346](https://github.com/wazuh/wazuh/issues/19346)).

![Community confirmation](../screenshots/03-community-false-positive-confirmation.jpg)
*Community reports confirming the `/usr/bin/diff` rootcheck hit as a known false
positive from the broad signature.*

## Step 3A — False Positive Response ✅

The file cleared every check, so: false positive, declared with evidence.

- Suppression options considered: hash-based suppression of the known-clean file
  would be ideal — but Wazuh's rootcheck module doesn't support it, and disabling
  rule 510 entirely is too risky (trading noise for blindness).
- Decision: **accepted the alert** and saved the clean SHA256 hash for future
  reference/comparison.

## Step 4 — Lessons Learned & Documentation ✅

- **Takeaway:** a high-severity alert does not always mean an actual crisis. The
  level 7 looked alarming; the evidence said otherwise.
- **Process validation:** the framework's first real-world run worked end to end —
  Categorize → Verify → Respond → Document produced a defensible verdict with an
  audit trail instead of a guess.
- **Detection note:** broad rootcheck regex signatures are a known noise source;
  worth tracking as a tuning candidate (hash-based allowlisting if/when the
  module supports it).

---

*A repeatable process is what separates the signal from the noise.*
