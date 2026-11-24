---
description: Open a maintenance window on a Zilean instance before a deploy, database migration or host maintenance. Pauses automation with a reason, then waits until no run is in flight, so the maintenance step starts on an idle scheduler. Other skills, such as a deploy skill, can call it as zilean:maintenance-window.
argument-hint: "<reason> [--wait <minutes>]"
allowed-tools:
  - mcp__plugin_zilean_zilean__zilean_pause
  - mcp__plugin_zilean_zilean__zilean_pause_status
  - Bash(sleep 30)
---

Open a maintenance window on the connected Zilean instance: pause, then wait
for running work to drain. This skill never resumes. The caller resumes
(step 6).

Input: a reason, optionally followed by `--wait <minutes>` at the end.

   $ARGUMENTS

- If the input ends with `--wait <minutes>`, that option is the wait limit and
  the value must be a positive whole number. Everything before `--wait` is the
  reason, verbatim. If `--wait` is present but its value is missing or is not
  a positive whole number, do not pause: stop and ask the caller.
- Without `--wait`, the whole input is the reason and the limit is 10 minutes.
  A trailing number is part of the reason, never a limit: "restart db 2" is
  the reason "restart db 2".

Follow these steps in order.

1. **Require a reason.** If the reason (the input without any `--wait`
   option) is empty or only whitespace, do not pause. Stop and ask the caller
   for a reason. A reason such as "deploy v1.2.3" is stored with the pause and
   shown to other operators.

2. **Pause.** Call `zilean_pause` once with that reason and nothing else:
   never pass `stop` or `stop_after_minutes`. Note `already_paused`.
   - If it is `true`, someone else's pause is in place and its reason and
     owner were kept. Tell the caller who set it and why (`paused_by`,
     `reason`), keep waiting, and remember this pause is not ours.
   - If the call fails with a tool error, report its text and stop.

3. **Wait for running work.** Call `zilean_pause_status` and read `running`.
   - `0`: go to step 5.
   - A positive number: wait, then check again.
   - `null`: the count is unknown, not zero. Keep waiting.
   - `paused` is `false`: someone resumed during the wait. Stop and report
     that the window is no longer in force.

   How to wait: run exactly `sleep 30` with the Bash tool between checks.
   This skill pre-approves that one command and nothing else from Bash. Time
   is counted in checks, not by a clock: allow two checks per minute of the
   limit (20 checks for the default 10 minutes, counting the first one). If
   Bash is not available or the command is refused, do not check in a tight
   loop: report the current `running` value and stop with the instance still
   paused.

4. **At the limit.** If work is still running when the checks run out, stop
   waiting. Report how many runs are still in flight (or that the count is
   unknown) and leave the instance paused. Never set a stop, even if asked:
   tell the caller to look at `/zilean:pause-status` and use Zilean's own stop
   controls if they decide to stop running work.

5. **Drained.** Otherwise report "drained": paused, nothing running. Hand back
   to the caller for the maintenance step.

6. **Resume (for the caller).** Tell the caller that when the maintenance step
   finishes or fails, they must resume with `/zilean:resume`, but only if this
   skill made the pause (`already_paused` was `false`). If the pause existed
   before, do not resume it: it belongs to someone else.

7. **Missing tool.** If a tool is not available, or a call fails with "unknown
   tool", report it and stop. For `zilean_pause`, the connection lacks the
   `settings:pause` permission, so nothing was paused. For
   `zilean_pause_status`, it lacks `jobs:read`, so the wait cannot be checked:
   say the instance is paused, if the pause call succeeded. Fixes: run `/mcp`,
   choose zilean, sign in again and approve "pause and resume", or ask an
   administrator to grant the permission. Do not try another tool or another
   MCP server.
