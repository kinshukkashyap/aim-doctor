---
description: Audit your own AIM tracking — what it captured, what it missed, and what your numbers actually are
---

# /aim-audit

Run a complete audit of this machine's AIM tracking and report it plainly.

## Do this

1. **Pull the data.** Run:
   ```bash
   /Applications/AIM.app/Contents/MacOS/aim-cli monthly-rollups-json --days 200 --org-tz Asia/Kolkata
   ```
   If `reconciliation` is missing from the output, AIM is older than 0.1.41 — say so and
   stop; the gap analysis needs that version.

2. **Report per month**: days captured, active hours, and how many of those days are
   `estimated` (reconstructed from prompt history rather than measured).

3. **Report the capture gaps** from `reconciliation`:
   - `missing_days` — AIM had nothing; 0.1.41 backfills these
   - `partial_days` — AIM has some of the day but prompt history shows substantially more.
     **State clearly that these are not corrected**, and quantify the shortfall in hours.
   - `consistent_days` — sources agree

4. **Compute their headline number** the way AIM does: total active seconds ÷ calendar days
   in the range. Show the arithmetic, not just the result.

5. **Check staleness.** Compare the most recent day in the data against today. If AIM has
   not synced recently, everything after the last sync is invisible to everyone else.

6. **Say which of these it is**, explicitly:
   - the number is right and here is why
   - the number is right but under-counted, and here are the days and the reason
   - something does not add up → offer `/aim-rca`

## Do not

- Do not explain from memory — run the command.
- Do not soften a low number. Report it.
- Do not present `estimated` hours as equivalent to measured ones. They have no token or
  cost data and are a floor, recovering roughly 85% of true time.
