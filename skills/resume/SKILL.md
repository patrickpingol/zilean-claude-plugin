---
description: Resume automation on the connected Zilean instance after a pause. Held work is released and scheduled fires start again.
disable-model-invocation: true
allowed-tools:
  - mcp__plugin_zilean_zilean__zilean_resume
---

Resume automation on the connected Zilean instance.

1. Call the `zilean_resume` tool (`mcp__plugin_zilean_zilean__zilean_resume`)
   exactly once. It takes no arguments.

2. Report the result. Use `was_paused`: if it is `false`, say the instance
   was not paused and nothing changed. If it is `true`, say automation is
   running again.

3. If the tool is not available, or the call fails with "unknown tool",
   say clearly that nothing was resumed, because this connection lacks the
   `settings:pause` permission. Possible causes: the user's role lacks it, it
   was not approved at sign-in, or the instance runs a Zilean release older
   than 4.4.0. To fix it, run `/mcp`, choose zilean, sign in again and approve
   "pause and resume", or ask an administrator to grant the permission to the
   role. Never claim success, never retry, and never try another tool or
   another MCP server.

4. If the call returns a tool error, report its text as it is and stop.
