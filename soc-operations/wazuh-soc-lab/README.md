# Wazuh SOC Lab — Nexus Freight Home Lab

A working blue-team lab built around **Wazuh SIEM/XDR**, used to practice security monitoring,
alert triage, and incident investigation with a repeatable process — not just tooling.

## Environment

| Component | Detail |
|---|---|
| SIEM / XDR | Wazuh (manager + dashboard) |
| Monitored host | `nexus-soc-01` (Wazuh manager name; single-agent lab) |
| Lab name | Nexus Freight home lab |
| Time window shown | Last 24 hours |

## What lives here

- [`docs/alert-investigation-framework.md`](docs/alert-investigation-framework.md) — the triage
  process used for every alert in this lab: Categorize → Verify → Respond → Document.
  Field-tested, not theoretical.
- [`investigations/`](investigations/) — real investigations run through the framework,
  each with the raw alert, the evidence checked, the verdict, and the reasoning.
- [`screenshots/`](screenshots/) — annotated evidence backing each investigation.

## Investigations so far

| Date | Alert | Verdict |
|---|---|---|
| 2026-09-28 | Wazuh rule 510 — trojaned `/usr/bin/diff` (level 7) | **False positive** — known overly-broad rootcheck regex signature ([wazuh/wazuh#19346](https://github.com/wazuh/wazuh/issues/19346)) |

## Lab notes

- During the investigation window the dashboard showed 4,764 total events with 4,472
  authentication failures and zero level-12+ alerts — a noisy environment, which is
  exactly why a repeatable triage process matters: severity alone doesn't tell you
  what's real.
- Sanitization rule for everything published here: no credentials, keys, tokens, or
  real public IPs. Screenshots are evidence, annotated with what each one proves.
