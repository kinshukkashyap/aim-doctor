---
description: Write up an AIM bug you have found, as a report you can send to Kinshuk
---

# /aim-rca

Use when the audit found something that **cannot be explained** by the metric design or by
known capture limits — i.e. a genuine candidate AIM bug.

## Before writing anything, rule out the known-and-expected

Do not file these as bugs. They are documented behaviour:

- Hours look low because the denominator is calendar days, including weekends and leave.
- A month is empty because AIM was not scanning then and Claude Code had already deleted
  transcripts older than ~30 days.
- A month shows codex-only because Claude history for that period was pruned before capture.
- `estimated` days carry no token or cost figures — that is deliberate, absent ≠ zero.
- `partial_days` stay under-counted — AIM is authoritative for days it has data for.
- Parallel agents multiply hours — that is the metric's intent.

## If it survives that, write the report

Write to `~/Desktop/aim-bug-<YYYY-MM-DD>-<short-slug>.md` containing:

- **What I expected, and what I saw** — concrete numbers, not impressions.
- **The evidence** — the actual command output that shows the discrepancy. Paste it.
- **What I ruled out** — which of the expected causes above you checked and eliminated.
- **Environment** — AIM version (`aim-cli --help` header or the app's About), macOS version,
  which tools are in use (Claude Code / codex / Warp).
- **Reproduction** — the exact commands, so it can be re-run on another machine.
- **What I could NOT verify** — be explicit about the limits of what you checked.

Then tell the user the file path and that they can send it to Kinshuk.

## ⚠️ Tell them what is in the report before they send it

A good report quotes real local detail — `prefs.json` contents, project paths, database
rows, excluded folder names. That is what makes it reproducible, and it is all the user's
own data on their own machine. **But the entire point is that they forward it to someone
else**, so they may share more than they intended.

Before handing over the file path, say plainly what went into it. Call out specifically:

- **Excluded project names and paths.** These are the things the user deliberately hid from
  tracking, so they are the most likely to be sensitive — other clients, side work, personal
  directories. A real report has already named `Documents/<client>` or similar.
- **Any absolute paths** revealing directory structure, usernames, or unrelated employers.
- **Repository and project names** not connected to the bug.

Then offer to redact: *"I can replace the excluded project names with placeholders if you'd
rather not share them — the bug reproduces either way."* Placeholders like `<excluded-1>`
keep the report valid, because what matters is that a day was excluded, not which project
it was.

**Never send or upload the report anywhere yourself.** Write the file, say what is in it,
and let the user decide.

## Rules

- **Every number in the report must come from a command you actually ran.** No estimates
  presented as measurements.
- If you are unsure whether it is a bug, say so in the report rather than asserting it.
- Never file a report to make the user feel vindicated. A wrong bug report wastes more time
  than a low number does.
