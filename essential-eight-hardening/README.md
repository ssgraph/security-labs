# Essential Eight Hardening — Windows Home Machine

Audit and harden a Windows 10/11 machine (or VM) against the
[ASD Essential Eight](https://www.cyber.gov.au/resources-business-and-government/essential-cyber-security/essential-eight)
mitigation strategies, documenting the baseline, every change made, and how each
change was verified.

## Objective

- Record an honest baseline of the machine before any changes.
- Work through each Essential Eight control: assess, harden, verify.
- Capture anything that broke along the way and how it was diagnosed.

## Environment

<!-- Fill in before starting -->
- Machine: (e.g. home desktop / VirtualBox VM)
- OS + version: (`winver`)
- Patch level: (`Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 5`)

## Baseline audit

Recorded **before** making any changes. Commands used are shown so the audit is reproducible.

| # | Control | How I checked | Baseline finding |
|---|---------|---------------|------------------|
| 1 | Application control | | |
| 2 | Patch applications | | |
| 3 | Configure MS Office macro settings | | |
| 4 | User application hardening | | |
| 5 | Restrict administrative privileges | `net localgroup Administrators` | |
| 6 | Patch operating systems | `Get-HotFix` | |
| 7 | Multi-factor authentication | | |
| 8 | Regular backups | | |

## Hardening steps

<!-- One section per control, as you complete it:
### Control N — <name>
- What the control is and why it matters
- What I changed (exact commands / menu paths)
- Verification: expected vs observed output
-->

## What broke and how I diagnosed it

<!-- The most important section. Symptom → hypothesis → test → root cause → fix. -->

## Lessons learned

<!-- Short, honest bullets. -->
