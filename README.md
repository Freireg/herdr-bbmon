# bbmon — BitBake build monitor for Herdr

Shows every running `bitbake` build in Herdr: which pane it is in, progress,
current task, whether an AI agent launched it, and a notification when it ends.

## Install

    herdr plugin install freireg/herdr-bbmon        # from GitHub, once pushed
    # or, from a local checkout:
    herdr plugin link /path/to/herdr-bbmon
    herdr plugin action invoke bbmon.start          # or restart herdr (startup hook)
    herdr plugin action invoke bbmon.setup-sidebar  # adds a "$bb" row to [ui.sidebar.agents]

Keybinding (already in ~/.config/herdr/config.toml):

    [[keys.command]]
    key = "prefix+shift+b"
    type = "plugin_action"
    command = "bbmon.list"

## What you see

| Where the build runs | Sidebar |
|---|---|
| plain shell pane | agent **bitbake** (yellow/working dot → done), label `⚙ 63% task 452/8961 · zlib-native:do_compile` |
| agent pane (hermes/claude/codex…) | agent keeps its own state; `$bb` row shows the build progress |
| end of build | `✔ succeeded 53s · 148 tasks` / `✖ FAILED … · 2 err` / `✖ interrupted`, plus a Herdr toast (kept for 15 min) |

`prefix+shift+b` popup: all running and recently finished builds with
workspace › tab › pane, user vs agent, build dir; `1-9` jumps to the pane,
`l` then a number opens the cooker log in `less`.

## How it works

* `/proc` scan every 2 s for BitBake **client** processes (`.../bin/bitbake`).
  `bitbake-server` daemonises to init, so only the client is tied to a pane.
* Parent chain of the client → pane shell PID (`pane.process_info`) → pane.
  Agent processes in the chain (or `AI_AGENT` in its env) = agent-launched.
* Progress: `<BUILDDIR>/tmp*/log/cooker/<machine>/<ts>.log`
  (`Running [setscene ]task N of M`). Before tasks start, knotty's footer on the
  pane screen (`Parsing recipes: 85%`) is used.
* Result: `Tasks Summary` + `ERROR:` lines; `Stopped build` or attempted < planned
  ⇒ interrupted.

## Commands

    python3 bbmon.py scan     # one-shot JSON of current builds (debug)
    python3 bbmon.py stop     # stop the watcher and clear its sidebar state

State/logs: `~/.local/state/herdr/plugins/bbmon/{bbmon.log,builds.json}`.
Tune with `BBMON_POLL_S`, `BBMON_DONE_HOLD_S`.

## Limits

* Builds started outside Herdr appear in the popup as "outside herdr", without a sidebar entry.
* Custom `TMPDIR` outside the build dir: no log progress (footer only).
* Linux only (/proc).
