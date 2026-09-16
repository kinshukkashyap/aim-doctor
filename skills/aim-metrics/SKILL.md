---
name: aim-metrics
description: Explain and debug a person's own AIM usage numbers. Use whenever someone asks why their AI hours, consistency, utilisation or leaderboard rank looks the way it does, why a month looks empty or low, why their number changed, whether AIM is tracking them correctly, or whether they have found a bug in AIM. Triggers on "why is my number low", "why am I ranked here", "why is April empty", "is AIM counting me correctly", "my hours look wrong", "AIM bug".
---

# Debugging your own AIM numbers

Your job is to get this person to the **actual cause** of what they are seeing, using data
from their own machine, and to be honest about which of three things it is:

1. **The number is right, and here is the arithmetic.**
2. **The number is right but incomplete, because AIM never captured some days** — and here
   is exactly which days and why.
3. **Something genuinely does not add up** — this may be an AIM bug, and you should offer
   to write it up (see `/aim-rca`).

Never guess which one it is. Get the data first.

## Step 1 — always get real data before saying anything

Do not explain from memory. Run the user's own AIM CLI, which ships inside the app:

```bash
/Applications/AIM.app/Contents/MacOS/aim-cli monthly-rollups-json --days 200 --org-tz Asia/Kolkata
```

That returns `rollups` (per month, with a `daily` map) and a `reconciliation` block.
If the binary is missing, AIM is not installed at the default path — ask where it is
rather than assuming.

**Requires AIM 0.1.41 or later.** If `reconciliation` is absent from the payload, say so
plainly: the capture-gap analysis is unavailable and they should update AIM first.

Useful raw sources on the same machine, if you need to go deeper:

| Source | What it is |
|---|---|
| `~/Library/Application Support/AIM/claude.db` | what AIM captured from Claude Code |
| `~/Library/Application Support/AIM/codex.db` | same for codex |
| `~/.claude/history.jsonl` | every Claude prompt, **never pruned** — the ground truth |
| `~/.claude/projects/**/*.jsonl` | Claude transcripts, **deleted after ~30 days** |

## Step 2 — the metrics, exactly as AIM computes them

Get these right. People usually disagree with the *definition*, not the arithmetic.

**AI work hours/day** — `total active_seconds ÷ calendar days in the selected range`.
- The denominator is **calendar days, not active days**. A month with 31 days divides by
  31 even if the person worked 8 of them. Weekends, leave and holidays all dilute it.
- This changed in 0.1.34. It used to divide by active days, which penalised people who
  worked 7 days a week at lower intensity.
- **Consequence worth stating out loud:** someone who does 40 focused hours in 5 days
  scores lower per-day than someone who does 40 hours spread over 20 days. That is the
  metric working as designed, not a bug.

**active_seconds** — the sum of gaps between consecutive turns *within a session*, counting
only gaps of **30 minutes or less** (`ACTIVE_GAP_THRESHOLD_MIN = 30`).
- It is **not** wall-clock. Leaving a session open overnight adds nothing.
- **Parallel sessions multiply.** Three agents running for one hour adds three hours. This
  is deliberate — it measures cumulative agent-time, not human-time. Someone who runs
  agents in parallel will legitimately score several times someone who does not.

**Utilisation %** — `active_seconds ÷ (calendar days × 8h)`, capped at 100.

**Consistency %** — `active days ÷ workdays (Mon–Fri)` in range, **capped at 100**. This is
the one metric no amount of parallelism can inflate — it only measures showing up.

⚠️ **Both Utilisation and Consistency are capped with `Math.min(100, …)`. If you compute
either by hand and get a figure above 100%, that is YOUR arithmetic missing the cap, not an
AIM bug.** Consistency exceeds 100% before capping for anyone who works weekends, because
the denominator counts only Mon–Fri. Do not report this as a defect — it was the first false
positive this plugin produced in testing.

## Step 3 — why days go missing, which is usually the real answer

This is the single most common cause of "my number is too low", and it is not the user's
fault or a calculation error.

**Claude Code deletes its own session transcripts after about 30 days** (`cleanupPeriodDays`,
default 30). AIM builds its history by scanning `~/.claude/projects`. So:

- If AIM was not running and scanning during a period, and more than ~30 days have passed,
  **those Claude transcripts are gone from the machine entirely.** No sync, update or
  re-scan can recover them.
- **codex sessions are not pruned the same way** — they commonly survive ~11 months. So a
  month showing codex-only activity often means "Claude was used too, but was not captured",
  not "they only used codex."
- AIM's own SQLite is a durable archive: once AIM has scanned a day, it keeps it even after
  Claude deletes the original.

**AIM 0.1.41 recovers part of this.** `history.jsonl` is never pruned, so AIM reconstructs
days it never captured and marks them `"estimated": true`. Those days carry hours, sessions,
turns and projects but **deliberately omit tokens, cost and cache** — absent, not zero, so
unknown stays distinguishable from zero.

Reconstructed time is a **floor, not an estimate**: it is derived from prompt timestamps only
and cannot see assistant turns filling long gaps, so it recovers roughly 85% of true measured
time. If someone's reconstructed month looks low, it is probably still under-counting.

## Step 4 — reading the reconciliation block

- `missing_days` — AIM has nothing, `history.jsonl` shows real activity. These are
  backfilled as `estimated` in 0.1.41.
- `partial_days` — AIM captured *some* of the day but `history.jsonl` shows much more.
  **These are NOT corrected.** AIM is treated as the authority for any day it has data for,
  so these stay under-counted by design. If a user's number looks low and they have many
  partial days, this is the honest explanation and it is a known limitation.
- `consistent_days` — the two sources agree.

## Step 5 — answering "why am I ranked here"

You usually cannot see other people's numbers, and should not try. What you can do:

- Compute their own number precisely and show the arithmetic.
- Identify how much of their total is `estimated` versus measured.
- Point out that **the leaderboard is only comparable between people running the same AIM
  version**. Someone on 0.1.41 has backfilled gaps; someone on an older build does not.
  A rank gap can be an artefact of who has updated, not of who did more work.
- Check how stale their own upload is. If their last sync is weeks old, everything after it
  is simply absent from what others see.

## Honesty rules

- **Never flatter the number.** If the data says they used AI for 20 minutes a day, say so.
- **Never invent a bug to make someone feel better**, and never dismiss a real discrepancy
  to defend the tool.
- Separate clearly: *"this is the metric working as designed and you may disagree with the
  design"* from *"this is broken."* The first is a product conversation with Kinshuk; the
  second is a bug report.
- If a number cannot be explained from the data, say that plainly and go to `/aim-rca`.
