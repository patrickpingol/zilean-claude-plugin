---
description: Pause all automation on the connected Zilean instance. Scheduled fires are skipped and queued work is held; running work finishes. Optional reason is stored with the pause.
argument-hint: "[reason]"
disable-model-invocation: true
allowed-tools:
  - mcp__plugin_zilean_zilean__zilean_pause
---

Pause automation on the connected Zilean instance.

1. Call the `zilean_pause` tool (`mcp__plugin_zilean_zilean__zilean_pause`)
   exactly once.
   - If the text after the command is not empty, pass it unchanged as the
     `reason` argument: the whole string as typed, with no quoting changes, no
     trimming and no rewording. The tool trims it and checks its length.
   - If the text is empty, call the tool with no arguments.
   - Never pass `stop` or `stop_after_minutes`. Running work is allowed to
     finish.

   The text to pass as the reason:

   $ARGUMENTS

2. Report the result:
   - `paused` and `already_paused`.
   - The stored `reason`, if any, and the `channel`.
   - `running`: the number of runs still in flight. `null` means unknown, not
     zero: say "unknown", never "0".
   - If `already_paused` is `true`, say that an existing pause was kept and
     the new reason was not stored. Name who set it and why, from
     `paused_by` and `reason`.

3. If the tool is not available, or the call fails with "unknown tool",
   say clearly that nothing was paused, because this connection lacks the
   `settings:pause` permission. Possible causes: the user's role lacks it, it
   was not approved at sign-in, or the instance runs a Zilean release older
   than 4.4.0. To fix it, run `/mcp`, choose zilean, sign in again and approve
   "pause and resume", or ask an administrator to grant the permission to the
   role. Never claim success, never retry, and never try another tool or
   another MCP server.

4. If the call returns a tool error (for example, a reason over 500
   characters), report its text as it is and stop.
