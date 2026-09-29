# Alert Investigation Framework

**Status:** v0.1 — field-tested 2026-09-28 (first live run:
[trojaned `/usr/bin/diff` → false positive](../investigations/2026-09-28-trojaned-usr-bin-diff-false-positive.md)).

A repeatable process for working a SIEM alert from fire to verdict. The point is
consistency: the same steps every time, so the verdict comes from evidence instead
of gut feeling. A high-severity alert is not automatically a crisis — the process
is what separates the signal from the noise.

## The process

### Step 1 — Categorize

Understand what the alert is actually about before touching anything.

- What is the flagged artifact, and what does it normally do?
- Who or what should ever legitimately touch it (root? the package manager? a service account)?
- What's the blast radius if the alert is real?

Example: a core system binary in `/usr/bin` should only ever be touched by root or
the package manager — and if it's truly trojaned, anything that executes it runs
attacker code. High stakes, so verification has to be airtight.

### Step 2 — Verify (three checks)

No verdict without evidence. Run all three:

1. **Local logs** — check package manager / system logs for any recent update,
   install, or modification touching the flagged file. An unexplained change is
   a red flag; a logged legitimate update is context.
2. **Local integrity** — run a package integrity check against the known-good
   package (e.g. `rpm -V`, `debsums`, AIDE baseline). Tampering shows up here
   if the file was actually modified.
3. **External threat intel** — look up the file hash (VirusTotal and similar),
   and check vendor/community references for known false positives. A 0/63
   clean with community confirmation of a bad signature is strong evidence.

All three checks must agree before calling it.

### Step 3A — False Positive Response

Taken when the file clears every Step 2 check.

- Declare the false positive **with the evidence attached** — verdict, checks run,
  and why the detector fired (e.g. an overly-broad regex signature).
- Suppression: prefer hash-based suppression of the known-clean file. If the
  detector doesn't support it (Wazuh's rootcheck module doesn't), **do not
  disable the rule entirely** — that's trading noise for blindness. Accept the
  alert and record the clean SHA256 hash for future comparisons.

### Step 3B — True Positive Response

Taken when any Step 2 check fails. **Not yet exercised in this lab** — branch
documented for completeness:

- Contain: isolate the host, block identified indicators.
- Eradicate & recover: remove the malicious artifact, rebuild from known-good
  media if trust in the host is lost, rotate exposed credentials.
- Then proceed to Step 4.

### Step 4 — Lessons Learned & Documentation

Close the loop on every investigation, whatever the verdict:

- Log what happened: the alert, the evidence checked, the verdict, the rationale.
- Write down the takeaway. (First run's takeaway: a high-severity alert does not
  always mean an actual crisis — the process earns its keep on the scary-looking
  ones.)
- Feed improvements back into detection: tuning notes, suppression candidates,
  runbook updates.

## Field test log

| Date | Alert | Verdict |
|---|---|---|
| 2026-09-28 | Wazuh rule 510 — trojaned `/usr/bin/diff`, level 7 | False positive — known broad rootcheck regex ([wazuh/wazuh#19346](https://github.com/wazuh/wazuh/issues/19346)) |
