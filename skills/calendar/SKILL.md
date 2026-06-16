---
name: calendar
description: Read upcoming interview events from the macOS Calendar.app via AppleScript, so interview dates/times can be pulled automatically instead of asked. Use when preparing for interviews, checking who is scheduled, or filling interview dates into Records.md / Quiz / Prep files.
user-invocable: true
---

# Interview Calendar

Read upcoming interview events from macOS Calendar.app.

## Requirements

- macOS Calendar.app with the interview events.
- Automation permission: the terminal/Claude process must be allowed to control Calendar (System Settings → Privacy & Security → Automation). The first `osascript` call triggers the prompt; if it errors with `-1743` / "Not authorized", ask the user to grant it.

## Step 1 — Scan for interview events

Run this `osascript` (default window: next 14 days). It scans **all** calendars and keeps events whose summary looks like an interview (面試 / interview) or contains a candidate name.

```bash
osascript <<'EOF' 2>&1
set output to ""
set d1 to current date
set d2 to d1 + (14 * days)
tell application "Calendar"
  repeat with c in calendars
    try
      set evs to (every event of c whose start date ≥ d1 and start date ≤ d2)
      repeat with e in evs
        set s to summary of e
        if s contains "面試" or s contains "interview" or s contains "Interview" then
          set output to output & ((start date of e) as string) & " | " & s & " | " & (name of c) & linefeed
        end if
      end repeat
    end try
  end repeat
end tell
return output
EOF
```

- To widen/narrow the window, change `14`.
- The summary convention seen in practice is `面試-{候選人}({面試官})`. Parse the candidate name out of the parentheses.

## Step 2 — Report

Present a table: 候選人 | 日期 | 時間 | (行事曆). Sort by date ascending.

Map each event to a candidate folder under `{recruitment_root}/Candidate/Ongoing/{name}` when possible.

## Notes

- Read-only. Never create/modify/delete calendar events.
- `osascript` event queries can be slow on large calendars; if it hangs, narrow the day window or filter to specific calendars by name.
- This skill only supplies dates. Feed them into `interview:review-resume` → `interview:quiz` → `interview:prep` → `interview:quiz-scoresheet` as usual.
