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
| 3 | Office macro settings | Office VBA execution policy | ⚠️ Weak default → ✅ [fixed](#step-1--disable-office-macros) |
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
- **What's actually listening:** worth knowing before touching a firewall.

  ```
  $ lsof -nP -iTCP -sTCP:LISTEN
  rapportd    *:58764   Apple Continuity (Handoff, AirDrop, Universal Clipboard)
  ControlCe   *:7000    AirPlay Receiver
  ControlCe   *:5000    AirPlay Receiver
  ```

  No SSH, no file sharing, no screen sharing — a clean result. All three are
  bound to `*`, though, so they're reachable by anything on the same network.
- **AirPlay Receiver scope: "Current User"** — checked in System Settings →
  General → AirDrop & Handoff, because Control Center is sandboxed and the
  setting can't be read from the CLI. I'd tightened this myself a few days
  before starting the lab, so it was already at the stricter of the two options
  and needed no change here — only devices signed into my own Apple ID are
  accepted.

  Worth noting the port stays open either way: that setting controls *who gets
  accepted*, not *whether the service listens*. A port scan can't tell you
  which mode you're in — a detail I'd have got wrong if I'd only looked at
  `lsof`.

## Where that leaves things

Two clear passes (OS patching, and the FileVault/SIP posture around it), two
outright fails (admin privileges, backups), a weak Office macro default, a
disabled firewall, and two items needing manual review. For a machine I'd have
described as "pretty secure" before starting, that's a useful dose of humility —
and exactly why you baseline before you harden.

## Hardening, one control at a time

Working through these easiest-win-first, most-disruptive-last:

1. ✅ Office macro policy — **done**, written up below
2. ✅ Application firewall — **done**, written up below
3. ⬜ MFA + browser settings — the manual checks
4. ⬜ Backups — configure a Time Machine destination
5. ⬜ Admin privileges — move daily work to a standard account

### Step 1 — Disable Office macros

**Before I touched anything, I checked whether I actually use macros.** No point
breaking a workflow to fix a theoretical risk. Four checks, all negative:

```
$ mdfind -name ".docm"    # and .xlsm, .xlsb, .pptm, .dotm, .xlam
(no results — no macro-enabled files anywhere on the disk)

$ ls ~/Library/Group\ Containers/UBF8T346G9.Office/User\ Content.localized/Startup.localized/*
(Word/Excel/PowerPoint startup folders empty — no personal macros installed)
```

Office's recent-documents records showed no macro-format files ever opened, and
the Add-Ins folder held nothing but Microsoft's stock language files. So
disabling macros outright costs me nothing.

**First attempt — and it failed:**

```
$ defaults write com.microsoft.Word VisualBasicMacroExecutionState -string DisabledWithoutWarnings
Could not write domain /Users/sunix/Library/Containers/com.microsoft.Word/Data/Library/Preferences/com.microsoft.Word; exiting
```

See [things that broke](#things-that-broke) for how I diagnosed that. Short
version: Office apps are sandboxed, and macOS blocks other processes from
writing into an app's container.

**What I did instead — a configuration profile.** This is how a managed fleet
would do it: the setting is delivered as *managed policy* rather than a user
preference, so it sits above the app container and can't be clicked off in
Word's settings. Profile is committed here:
[`artifacts/disable-office-macros.mobileconfig`](artifacts/disable-office-macros.mobileconfig).

It sets `VisualBasicMacroExecutionState = DisabledWithoutWarnings` for Word,
Excel and PowerPoint, and I installed it via System Settings → General →
Device Management.

One detail that would have silently broken it: I pulled the bundle IDs out of
the apps themselves rather than typing what looked right.

```
$ defaults read "/Applications/Microsoft PowerPoint.app/Contents/Info.plist" CFBundleIdentifier
com.microsoft.Powerpoint
```

Lowercase "p". Word and Excel keep their capitals, PowerPoint doesn't. Since
the payload type *is* the preference domain, `com.microsoft.PowerPoint` would
have installed perfectly happily and applied to nothing.

**Verification.** The obvious check gives the wrong answer:

```
$ defaults read com.microsoft.Word VisualBasicMacroExecutionState
The domain/default pair ... does not exist
```

That's `defaults` reading only the app container — it doesn't consult managed
preferences at all. `profiles list` was equally misleading ("no configuration
profiles installed for user") because the profile is System-scope, not
user-scope. Two commands, two confident wrong answers.

The check that actually counts asks the same API the apps use:

```
$ osascript -l JavaScript -e '...CFPreferencesCopyAppValue / CFPreferencesAppValueIsForced...'
com.microsoft.Word       => DisabledWithoutWarnings  (managed/forced: true)
com.microsoft.Excel      => DisabledWithoutWarnings  (managed/forced: true)
com.microsoft.Powerpoint => DisabledWithoutWarnings  (managed/forced: true)
```

`forced: true` is the bit that matters — the value comes from managed policy,
so it isn't something I can accidentally turn off later. And the managed plists
are on disk where they should be:

```
$ ls /Library/Managed\ Preferences/
com.microsoft.Excel.plist  com.microsoft.Powerpoint.plist  com.microsoft.Word.plist
```

**Result:** control 3 goes from "prompts me to make a security decision at the
worst possible moment" to "macros don't run, no prompt, and I can't casually
override it".

### Step 2 — Enable the application firewall

Not one of the eight, but it came out of the baseline as disabled and it's
cheap to fix.

**What this firewall actually is — and isn't.** It's an *application-layer
inbound* firewall: it decides which apps may accept incoming connections. Two
things it does not do, both of which people assume it does:

- **It doesn't filter outbound traffic.** Malware already running and calling
  home is completely unaffected.
- **It doesn't close ports belonging to signed Apple software.** "Automatically
  allow built-in signed software" is on by default, so the AirPlay and
  Continuity listeners from the baseline stay reachable.

So turning it on didn't change my exposure on day one. Its value is over
*future* third-party apps, which now have to ask before accepting inbound
connections — plus stealth mode, below.

**Before:**

```
$ /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
Firewall is disabled. (State = 0)
$ /usr/libexec/ApplicationFirewall/socketfilterfw --getstealthmode
Firewall stealth mode is off
```

**What I changed.** Firewall on via System Settings → Network → Firewall, then
stealth mode:

```
$ sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode on
```

I also wrote a configuration profile for this
([`artifacts/enable-application-firewall.mobileconfig`](artifacts/enable-application-firewall.mobileconfig))
since that's how a managed fleet would deploy it, and it's kept here as the
reusable version. In the end I applied the change through the GUI and CLI
instead, so the profile is untested on this machine — flagging that rather than
implying it's proven.

**After:**

```
$ /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
Firewall is enabled. (State = 1)
$ /usr/libexec/ApplicationFirewall/socketfilterfw --getstealthmode
Firewall stealth mode is on
```

**Behavioural proof, not just a flag.** Stealth mode means the machine stops
answering pings and silently drops probes to closed ports, so it doesn't
announce itself on a network it doesn't control. Pinging my own LAN address:

```
$ ping -c 2 192.168.0.11
Request timeout for icmp_seq 0
2 packets transmitted, 0 packets received, 100.0% packet loss
```

That's the control working, confirmed by behaviour rather than by reading back
the setting I just wrote.

**⚠️ Trade-off I'm accepting deliberately — read this before troubleshooting.**
This Mac will no longer respond to ICMP echo from *anything*, including my own
lab gear. When I'm doing CCNA work and a ping from a switch or a VM to this
laptop times out, **that is stealth mode, not a routing problem.** Writing it
down here because in six months I'd have burned an hour on it. To temporarily
reverse it:

```
$ sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setstealthmode off
```

**One thing I couldn't verify.** The profile sets `EnableLogging`, but this
version of `socketfilterfw` doesn't expose `--getloggingmode` (it's absent from
the usage output), and `/var/log/appfirewall.log` doesn't exist yet. Logging is
therefore unconfirmed — not claiming it as done.

## Things that broke

### `defaults write` couldn't touch Office's preferences

**Symptom:** `Could not write domain .../com.microsoft.Word; exiting`.

**What I thought it was:** a typo in the domain, or needing sudo.

**How I tested that:** tried listing the folder directly, and writing a
throwaway key to the same domain:

```
$ ls -la ~/Library/Containers/com.microsoft.Word/Data/Library/Preferences/
ls: ...: Operation not permitted
```

"Operation not permitted" rather than "permission denied" is the tell — that's
macOS privacy protection, not Unix file permissions. I confirmed it wasn't my
terminal environment by re-running the same listing with my shell's sandbox
disabled: identical result.

**Root cause:** Word, Excel and PowerPoint are sandboxed apps. macOS protects
each app's container in `~/Library/Containers/` from any process that isn't
that app or doesn't hold Full Disk Access. `defaults write` is neither.

**Fix:** deliver the setting as managed policy via a configuration profile
instead. I deliberately did *not* grant Full Disk Access to my terminal to
force the command through — handing a shell blanket access to every protected
folder on the machine is a much bigger privilege than the problem justified,
and it would contradict the least-privilege change coming in step 5.

### An audit command that lied to me by omission

While writing the baseline I ran the macro check with `2>/dev/null` and a
fallback message, so a permission failure would have been indistinguishable
from "the setting isn't configured". I re-ran it with errors visible and macOS
genuinely reported the key doesn't exist, so the finding stood — but it was
luck, not method. Don't suppress stderr on a command whose entire job is to
tell you the truth about a machine.

## Lessons learned

- Baseline before you harden. My guesses about my own machine were wrong in
  both directions.
- Mapping a Windows-centric framework onto another OS forces you to learn the
  *intent* of each control, not the checkbox.
- Check whether a control is even in use before enforcing it — the macro search
  took two minutes and turned "this might break something" into a known-zero
  cost.
- "The policy deployed" and "the policy took effect" are different claims, and
  only the second matters. Verify at the layer where the control actually
  lives.
- A hardening step that requires weakening something else (Full Disk Access for
  a terminal) usually means there's a better route.
- Know what a control actually covers before claiming it. Turning the firewall
  on felt like progress, but inbound-only filtering with signed software
  auto-allowed changed nothing about what was reachable that day.
- Prove controls by behaviour where you can. "Stealth mode is on" is a setting;
  "my own ping to this machine now times out" is evidence.
- Write down the side effects you've accepted, not just the changes you made.
  Stealth mode will make a future troubleshooting session lie to me unless the
  reason is recorded somewhere I'll look.
