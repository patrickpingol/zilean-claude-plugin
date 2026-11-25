# Zilean plugin for Claude Code

A Claude Code plugin for [Zilean](https://zilean.dev/), a
self-hosted job scheduler. It connects Claude Code to one Zilean instance over
MCP with browser sign-in (OAuth), adds three slash commands to pause and
resume the instance's automation, and adds a skill that opens a maintenance
window: it pauses, then waits until no run is in flight.

No token is stored anywhere. You enter the instance's base URL once,
and a browser window opens for sign-in.

## Requirements

- Zilean 5.0.0 or later. Earlier releases do not let a signed-in connection
  pause or resume; the commands then report that the permission is missing.
- MCP enabled on the instance.
- `PUBLIC_BASE_URL` set on the instance, and the instance not in demo mode.
  Without both, OAuth sign-in does not work. See
  [Connecting a client](https://zilean.dev/guides/mcp/#connecting-a-client).
- A Zilean user whose role holds `settings:pause` and `jobs:read`.
  `settings:pause` is shown on the sign-in consent page; approve "pause and
  resume". The connection holds the permissions you approve that your role
  also has.

## Install

```
/plugin marketplace add patrickpingol/zilean-claude-plugin
/plugin install zilean@zilean
```

Claude Code asks for the **Zilean base URL**. Enter the instance's address with
`https://` and no trailing slash or path, for example
`https://zilean.example.com`. The plugin adds `/mcp`, and a trailing slash
would make the address `https://zilean.example.com//mcp`, which fails. Do not
add a token.

Then sign in:

1. Run `/mcp` and choose **zilean**.
2. Choose to authenticate. A browser window opens on your Zilean instance.
3. Sign in and approve the access it asks for, including "pause and resume".

To change the URL later, open `/plugin`, choose the plugin and use
**Configure options**.

## Commands and skill

| Name | What it does | Model can invoke it |
| --- | --- | --- |
| `/zilean:pause [reason]` | Pauses automation. The text after the command is stored as the reason. Running work finishes. | No |
| `/zilean:resume` | Resumes automation. Says if the instance was not paused. | No |
| `/zilean:pause-status` | Shows whether it is paused, who paused it and why, any pending stop, and how many runs are in flight. | Yes |
| `/zilean:maintenance-window <reason> [--wait <minutes>]` | Pauses with the reason, then waits until nothing is running (default limit 10 minutes; `--wait` sets it, as a positive whole number at the end). Never resumes: you resume with `/zilean:resume` when the maintenance step finishes or fails, unless the pause was already in place before. | Yes |

The model cannot pause or resume on its own: only you can type
`/zilean:pause` and `/zilean:resume`. The maintenance skill pre-approves only
pause and the status read, never resume, so other skills (a deploy skill, for
example) can call it as `zilean:maintenance-window` and still leave the resume
to you.

Everything before `--wait` is the reason, verbatim, so a trailing number is
never taken as a limit: `/zilean:maintenance-window restart db 2` has the
reason "restart db 2", and `/zilean:maintenance-window restart db --wait 2`
waits at most 2 minutes. If `--wait` has no positive whole number after it,
the skill asks instead of guessing.

The maintenance skill waits by running `sleep 30` between status checks; it
pre-approves exactly that command and no other Bash command. At the wait limit it
reports how many runs are still in flight and leaves the instance paused. It
never stops running work; use `/zilean:pause-status` and Zilean's own stop
controls if you decide to.

For deploy automation, sign in as a user whose role holds only
`settings:pause` and `jobs:read`, so the connection can do nothing else.

## Good to know

- **Remove any manually added zilean MCP server first.** Claude Code
  de-duplicates MCP servers by URL, and a manual entry wins over a plugin's.
  If one points at the same endpoint, the plugin's server does not start and
  its commands cannot find their tools. Run `claude mcp remove zilean`, or
  remove the entry from your settings.
- **OAuth only.** This plugin does not use static MCP tokens. To connect with
  a token, follow [Issuing a token](https://zilean.dev/guides/mcp/#issuing-a-token)
  in the Zilean docs instead.
- **One instance per machine.** The plugin has a single base URL. To
  talk to a second instance, add it manually as a separate MCP server.
- **Local sessions only.** Cloud sessions (claude.ai/code) do not load
  plugins installed on your machine.

## Documentation

- [Install the Claude Code plugin](https://zilean.dev/guides/mcp/#install-the-claude-code-plugin)
- [MCP](https://zilean.dev/guides/mcp/): connecting clients, scopes and tools
- [Pausing the instance](https://zilean.dev/guides/mcp/#pausing-the-instance)
  and [Global pause](https://zilean.dev/guides/global-pause/): what a pause
  stops, what it doesn't, and stopping running work

## Releases

The plugin version tracks Zilean's: version 5.0.0 of the plugin matches Zilean
5.0.0. Tags are plain `vX.Y.Z` and match the Zilean release. Work happens on
`development` and moves to `main` by fast-forward.

## Verified live

Checked with Claude Code 2.1.331 against a Zilean instance that offers the
`settings:pause` sign-in scope:

- [x] `${user_config.url}` is substituted into the MCP server URL.
- [x] `allowed-tools` pre-approves the plugin's MCP tool names
      (`mcp__plugin_zilean_zilean__zilean_pause`, `..._zilean_resume`,
      `..._zilean_pause_status`): the three commands and the maintenance skill
      ran with no permission prompt.
- [x] The OAuth connection is bound to the user who approves the sign-in: the
      pause records that user as its actor, on the `mcp` channel, with the
      client "Claude Code" in the audit log.
- [x] `/zilean:pause` and `/zilean:resume` are not offered to the model.

Not yet verified:

- [ ] `allowed-tools: Bash(sleep 30)` pre-approves the wait command, so the
      maintenance skill raises no permission prompt while waiting (the live
      check had nothing running, so no wait happened).
- [ ] A base URL with `/mcp` appended by the plugin
      (`${user_config.url}/mcp`) connects and signs in. The live check used a
      full endpoint URL.
- [ ] What Claude Code does when the URL is not set.
- [ ] The failure a user without `settings:pause` sees from `/zilean:pause`.

## License

MIT. See [LICENSE](LICENSE).
