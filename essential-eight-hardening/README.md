# Essential Eight: auditing my Mac

The [Essential Eight](https://www.cyber.gov.au/resources-business-and-government/essential-cyber-security/essential-eight)
comes up in basically every week of my course, but all the material assumes
Windows — and my daily machine is a MacBook Pro M1. So instead of skipping the
hands-on part, I'm mapping each of the eight controls onto macOS and auditing my
own machine against them. That turned out to be a better exercise than following
a Windows checklist, because for every control I had to work out what it's
actually *for* before I could find the macOS equivalent.

(A proper Windows version on a cloud VM is planned as a follow-up lab.)

The approach is deliberately boring:

1. Baseline first. Record the state of the machine before touching a single
   setting, with the exact command used for each check.
2. Then one control at a time — harden it, and prove the change stuck by
   checking again afterwards.
3. If something breaks (and something always breaks), that gets written up too.

## The machine

- MacBook Pro (Apple M1, `arm64`)
- macOS 26.5.2, build 25F84 (`sw_vers`)
- Update history shows regular OS updates going back years (`softwareupdate --history`)

## Baseline — audited 20 July 2026

Verdict summary first, details underneath:

| # | Control | macOS equivalent I checked | Verdict |
|---|---------|---------------------------|---------|
| 1 | Application control | Gatekeeper | ⚠️ Partial |
| 2 | Patch applications | App Store auto-update | ⚠️ Partial |
| 3 | Office macro settings | Office VBA execution policy | ⚠️ Weak default |
| 4 | User application hardening | Browser/app settings | ❓ Not yet assessed |
| 5 | Restrict admin privileges | Membership of the `admin` group | ❌ Fail |
| 6 | Patch operating systems | Software Update automation | ✅ Pass |
| 7 | Multi-factor authentication | Apple ID two-factor | ❓ To verify manually |
| 8 | Regular backups | Time Machine | ❌ Fail |

### 1. Application control — Gatekeeper

**What it's for:** stopping unapproved executables from running at all. On
Windows this is AppLocker/WDAC allowlisting; macOS's closest built-in is
Gatekeeper, which blocks apps that aren't signed and notarised by Apple.

```
$ spctl --status
assessments enabled
```

**Verdict: partial.** Gatekeeper is on, which is good — but it's a
signing check, not an allowlist. Any signed/notarised app runs, and a user can
still right-click → Open their way past it. Honest score under the Essential
Eight's intent: better than nothing, well short of application control.

### 2. Patch applications — App Store auto-update

**What it's for:** attackers mostly exploit *known* holes in apps like browsers
and Office. Patching applications quickly closes them.

```
$ defaults read /Library/Preferences/com.apple.commerce AutoUpdate
1
```

**Verdict: partial.** App Store apps update themselves. But my Microsoft Office
apps come from Microsoft (updated via Microsoft AutoUpdate), and anything
installed outside the App Store isn't covered by this — needs its own check
in the hardening phase.

### 3. Office macro settings

**What it's for:** macros in Office documents are one of the most common ways
malware gets a foothold — a booby-trapped spreadsheet that runs code the moment
you enable it. The Essential Eight wants macros blocked, or at minimum blocked
from the internet.

Office for Mac is installed (Word, Excel, PowerPoint, Outlook), so this control
applies to me:

```
$ defaults read com.microsoft.Word VisualBasicMacroExecutionState
(key not set)
```

Same for Excel and PowerPoint. **Verdict: weak default.** With the key unset,
Office falls back to "prompt the user when a macro-enabled file opens" — which
means the security decision lands on me at the exact moment I'm curious about
an attachment. That's precisely the scenario this control exists to remove.

### 4. User application hardening

**What it's for:** trimming attack surface in the apps that touch untrusted
content all day — mainly the browser (blocking ads/scripts from hostile pages,
disabling legacy tech like Java in the browser).

**Verdict: not yet assessed.** Browser settings need a manual review; parked
for the hardening phase.

### 5. Restrict administrative privileges

**What it's for:** malware runs with the privileges of the user who ran it. If
your everyday account is an administrator, one bad click hands over the whole
machine. Daily work should happen in a standard account, with admin credentials
only entered when something legitimately needs them.

```
$ dscl . -read /Groups/admin GroupMembership
GroupMembership: root sunix
```

**Verdict: fail.** My daily account is an administrator. (There's a second,
standard account on the machine, but it's not the one I live in.) This is the
finding that stung — it's the exact thing I'd flag on someone else's machine.

### 6. Patch operating systems

**What it's for:** same logic as patching apps, but for the OS itself — kernel
and system vulnerabilities are the crown jewels for attackers.

```
$ sw_vers          → macOS 26.5.2 (current)
$ defaults read /Library/Preferences/com.apple.SoftwareUpdate AutomaticallyInstallMacOSUpdates
1
$ defaults read /Library/Preferences/com.apple.SoftwareUpdate CriticalUpdateInstall
1
```

**Verdict: pass.** OS is current, macOS updates and rapid security responses
both install automatically, and the update history shows it's actually been
happening, not just configured.

### 7. Multi-factor authentication

**What it's for:** a stolen password on its own shouldn't be enough to get in.
On a personal Mac the account that matters most is the Apple ID — it controls
iCloud, Find My, and password sync.

**Verdict: to verify manually.** There's no clean command-line check for Apple
ID two-factor; it needs eyes on System Settings → Apple ID → Sign-In &
Security. On the list for the hardening phase.

### 8. Regular backups

**What it's for:** the control that saves you when everything else fails —
ransomware, theft, or a dead SSD. No backup means every other mistake is
permanent.

```
$ tmutil destinationinfo
tmutil: No destinations configured.
```

**Verdict: fail.** No Time Machine destination at all, so no local backups
exist. (Some data lives in iCloud, but sync is not a backup — a bad change
syncs everywhere.)

### Things I checked while I was in there

Not Essential Eight controls, but part of an honest picture of the machine:

- **FileVault: on** (`fdesetup status`) — full-disk encryption, so a stolen laptop is a brick, not a data breach. Good.
- **System Integrity Protection: enabled** (`csrutil status`) — stops even root from tampering with system files. Good.
- **Application firewall: disabled** (`socketfilterfw --getglobalstate`, state 0) — nothing filtering inbound connections. Going on the fix list.

## Where that leaves things

Two clear passes (OS patching, and the FileVault/SIP posture around it), two
outright fails (admin privileges, backups), a weak Office macro default, a
disabled firewall, and two items needing manual review. For a machine I'd have
described as "pretty secure" before starting, that's a useful dose of humility —
and exactly why you baseline before you harden.

## Hardening, one control at a time

Coming next, in this order — easiest wins first, most disruptive change last:

1. Office macro policy — set macros to disabled
2. Application firewall — enable it
3. MFA + browser settings — the manual checks
4. Backups — configure a Time Machine destination
5. Admin privileges — move daily work to a standard account

Each one gets: what I changed, the exact command, and before/after proof.

## Things that broke

Nothing yet — but the hardening phase hasn't started. If this section ends up
empty I'll be suspicious of myself.

## Lessons learned

- Baseline before you harden. My guesses about my own machine were wrong in
  both directions.
- Mapping a Windows-centric framework onto another OS forces you to learn the
  *intent* of each control, not the checkbox.
