---
description: Show whether automation on the connected Zilean instance is paused, who paused it and why, any pending stop, and how many runs are still in flight.
allowed-tools:
  - mcp__plugin_zilean_zilean__zilean_pause_status
---

Show the automation pause status of the connected Zilean instance.

1. Call the `zilean_pause_status` tool
   (`mcp__plugin_zilean_zilean__zilean_pause_status`) once. It takes no
   arguments.

2. Report:
   - `paused`: whether automation is paused.
   - If paused: the `channel`, the `reason`, who paused it (`paused_by`) and
     when (`paused_at`).
   - Any pending stop of running work: `stop_mode` and `stop_at`. If there is
     none, say there is no pending stop.
   - `running`: the number of runs still in flight. `null` means unknown, not
     zero: say "unknown", never "0".

3. If the tool is not available, or the call fails with "unknown tool",
   say clearly that the status could not be read, because this connection
   lacks the `jobs:read` permission. To fix it, run `/mcp`, choose zilean and
   sign in again, or ask an administrator to grant the permission to the
   user's role. Never guess the status, and never try another tool or another
   MCP server.

4. If the call returns a tool error, report its text as it is and stop.
