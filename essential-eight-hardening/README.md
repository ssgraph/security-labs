# Essential Eight: auditing my own machine

The [Essential Eight](https://www.cyber.gov.au/resources-business-and-government/essential-cyber-security/essential-eight)
comes up in basically every week of my course, so instead of just memorising the
list I decided to point it at my own Windows machine. How many of the eight
controls would my daily driver actually pass? My guess going in: not many.

The approach is deliberately boring:

1. Baseline first. Write down the state of the machine before touching a single
   setting, with the exact command I used for each check.
2. Then one control at a time — harden it, and prove the change stuck by
   checking again afterwards.
3. If something breaks (and something always breaks), that gets written up too.

## The machine

*Filling this in before I start:*

- Machine:
- OS + version (`winver`):
- Recent patches (`Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 5`):

## Baseline

The point of logging the command next to each finding is that the audit itself
should be repeatable — anyone can run the same checks on their own machine and
compare.

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

## Hardening, one control at a time

Each control gets its own section as I get to it: what it's actually for, what
I changed (real commands and menu paths, not "configure the setting"), and the
before/after proof that it worked.

## Things that broke

Kept honest. Symptom, what I thought it was, how I tested that, what it actually
was, and the fix. If this section ends up empty I'll be suspicious of myself.

## Lessons learned

Added as I go.
