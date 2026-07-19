# Hardening my Windows machine against the Essential Eight

The [ASD Essential Eight](https://www.cyber.gov.au/resources-business-and-government/essential-cyber-security/essential-eight)
is Australia's baseline set of mitigation strategies, and it comes up constantly in
my course. Rather than just reading about it, I'm auditing my own machine against
all eight controls, fixing what I find, and writing down exactly what I did.

The plan:

1. Record an honest baseline **before** touching anything.
2. Go through each control one at a time — check it, harden it, verify the change
   actually took effect.
3. Keep notes on anything that breaks, because that's where the real learning is.

## The machine

<!-- filling this in before I start -->
- Machine:
- OS + version (`winver`):
- Recent patches (`Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 5`):

## Baseline — where things stood before I changed anything

I'm noting the exact command or menu path for each check so the audit itself is
reproducible, not just the fixes.

| # | Control | How I checked | What I found |
|---|---------|---------------|--------------|
| 1 | Application control | | |
| 2 | Patch applications | | |
| 3 | Office macro settings | | |
| 4 | User application hardening | | |
| 5 | Restrict admin privileges | `net localgroup Administrators` | |
| 6 | Patch operating systems | `Get-HotFix` | |
| 7 | Multi-factor authentication | | |
| 8 | Regular backups | | |

## Hardening, control by control

One section per control as I get to it: what the control is for, what I changed
(exact commands or menu paths), and how I verified it — expected vs what I
actually saw.

## What broke and how I worked it out

Honest notes only. Symptom, what I suspected, how I tested it, what the actual
cause was, and the fix.

## Lessons learned

Short and honest, added as I go.
