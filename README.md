# aim-doctor

A Claude Code plugin that lets anyone interrogate their own AIM numbers and get to the real
cause, instead of asking Kinshuk.

It answers three kinds of question honestly:

- **"Why is my number low?"** — shows the arithmetic, including the parts of the metric
  people tend to disagree with (hours divide by calendar days; parallel agents multiply).
- **"Why is this month empty?"** — Claude Code deletes its own transcripts after ~30 days,
  so anything AIM did not scan in time is gone. It shows exactly which days and why.
- **"Is this a bug?"** — rules out the documented behaviours first, and only then writes a
  reproducible report you can send on.

## How it works

It does **not** reimplement AIM's maths. It calls the `aim-cli` binary already inside the
installed app:

```
/Applications/AIM.app/Contents/MacOS/aim-cli monthly-rollups-json --days 200 --org-tz Asia/Kolkata
```

So the plugin cannot drift from the app. Everything is local — no credentials, no network,
no access to anyone else's data.

**Requires AIM 0.1.41+** for the capture-gap analysis (`reconciliation`).

## Install

```
/plugin marketplace add ~/Documents/Personal/Code/aim/plugin
/plugin install aim-doctor@aim-tools
```

## Use

- `/aim-audit` — full audit of what was captured, missed, and what the numbers are
- `/aim-rca` — write up a genuine bug as a shareable report
- Or just ask: *"why are my AIM hours lower than I expected?"*
