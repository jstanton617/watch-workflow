# watch-workflow

Watch every subagent of a Claude Code workflow at work, live, in its own
Ghostty split.

When a workflow runs a dozen reviewers at once, the final answer isn't enough:
you want to see what each one read, ran and concluded. `watch-workflow` turns
a Ghostty terminal into the run's status board:

- **one tab per phase** of the workflow, named for the phase;
- **one split per agent**, added as the agent starts, in rows `--cols` wide;
- **a status bar** on each pane's top line: the agent, its phase and state, its
  tool calls and its time;
- **the agent's transcript** below it, fitted to the pane: its text, each tool
  call, and the first lines of each result, with a count of the rest.

When an agent's result reaches the workflow's journal, its bar says done and
the pane stays open.

## Install

It's one standard-library Python file. Put it in a directory on your `PATH`,
such as `~/.local/bin`:

```bash
mkdir -p ~/.local/bin
```

```bash
curl -fsSL https://raw.githubusercontent.com/jstanton617/watch-workflow/main/watch-workflow -o ~/.local/bin/watch-workflow
```

```bash
chmod +x ~/.local/bin/watch-workflow
```

`~/.local/bin` isn't on macOS's default `PATH`. If `watch-workflow` isn't found,
add `export PATH="$HOME/.local/bin:$PATH"` to your `~/.zshrc`.

## Requirements

- macOS, with [Ghostty](https://ghostty.org) 1.3 or later: it drives Ghostty's
  AppleScript API, so macOS may ask you to let your terminal control Ghostty.
- Python 3.9 or later.
- Claude Code, running workflows. It's tested with Claude Code 2.1.310.

## Use

Run it directly in a Ghostty window while a workflow runs, not inside tmux or
over SSH: it checks that it's in Ghostty. Its tabs open in Ghostty's front
window, so keep that window in front as it starts.

```
watch-workflow                the newest run, in any project
watch-workflow wf_d557        a run by its id, or a prefix of it, or its directory
watch-workflow --list         the recent runs
watch-workflow --cols 3       columns per phase's tab (default: one per 100 characters)
watch-workflow --lines 5      lines of each tool result a pane shows (default 3, 0 for all)
watch-workflow --no-links     no links on the [+N lines] counts
watch-workflow --dry-run      print each osascript call instead of making it; it still follows the run
watch-workflow --self-test    check the parsing, layout and formatting offline
```

Ctrl-C stops the board. The panes are ordinary Ghostty splits and stay open.

**A cut result's `[+N lines]` is a link.** Cmd-click it, and the whole result
opens in your browser. The board serves each result from the agent's
transcript when its link is clicked, on `127.0.0.1` at a port of its own,
behind a random token, and only while it runs. Nothing is written to disk for
it, and once you stop the board the links stop working.

## What it reads

Everything it reads is local, and it changes none of it:

```
~/.claude/projects/<project>/<session>/subagents/workflows/wf_<id>/
    journal.jsonl        a "started" line per agent (its id, label and phase),
                         then a "result" line with what it returned
    agent-<id>.jsonl     that agent's transcript, written as it works
```

and the workflow's name, from `<session>/workflows/scripts/<name>-wf_<id>.js`.
`CLAUDE_CONFIG_DIR` moves `~/.claude`, as it does for Claude Code.

These are Claude Code's own files, and their shapes aren't a published
interface. A new Claude Code version can change them, and the board with them.

## License

MIT. See [LICENSE](LICENSE).
