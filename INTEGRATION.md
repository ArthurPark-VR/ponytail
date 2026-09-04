# Integration setup

Three things are wired together in this workspace, on branch
`claude/ponytail-caveman-integration-jp6yet`:

| Layer | Repo | What it governs |
| --- | --- | --- |
| Prose style | `caveman` | How the agent talks |
| Code volume | `ponytail` | What the agent builds |
| Workflow | `skills` (fork of `mattpocock/skills`) | Which process the agent runs |

Nothing here survives a container restart, so it has to be re-run per session.

## Re-activate, one command each

```bash
node /home/user/caveman/src/hooks/caveman-activate.js < /dev/null
node /home/user/ponytail/hooks/ponytail-activate.js < /dev/null
bash /home/user/skills/scripts/link-skills.sh
```

The two `*-activate.js` hooks print their ruleset on stdout. Claude Code injects
SessionStart hook stdout as system context, so running them by hand puts the
same rules in play. They also write mode flags to `$CLAUDE_CONFIG_DIR`
(`.caveman-active`, `.ponytail-active`), both defaulting to `full`.

`link-skills.sh` symlinks all 37 non-deprecated skills into `~/.claude/skills`
and `~/.agents/skills`. It is the repo's own dev script, not a supported
installer, and must not be modified. Symlinks point back into the clone, so a
`git pull` there refreshes every installed skill.

## Making it automatic

A `SessionStart` hook in `.claude/settings.json` would run the linker on its
own. It is not committed here: writing a file that auto-executes a script at
session start is a real security decision, and it belongs to whoever owns the
machine, not to the agent. Add it deliberately if you want it.

Note that the `skills` repo cannot hold such a hook at all: its `.gitignore`
excludes `.claude`, so anything placed there is untracked.

## Known conflicts

See the audit in the session that produced this file. The short version:

1. `caveman` and `ponytail` both want the single `statusLine` key in
   `settings.json`. Only one can own it; the loser nudges every session.
2. `normal mode` deactivates both of them at once. Use `stop caveman` or
   `stop ponytail` to turn off exactly one.
3. The `skills` fork ships a `code-review` skill whose name collides with the
   built-in one, so the fork's version stays shadowed and unreachable.
4. `tdd` (from the fork) and `ponytail` disagree about how much test code is
   warranted. `ponytail` already carves out an exception for anything the user
   explicitly asks for, which is what settles it in practice.
