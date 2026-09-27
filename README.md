# cctree

Show the Claude Code sessions of a project as a **conversation tree**, and resume, fork or "rewind" any of them without touching the original.

When you fork a session (`--fork-session`, `/branch`), Claude Code copies the parent's messages into the new session file and keeps their uuids. `cctree` merges all session files of a project by uuid, which gets the branching structure back.

## Requirements

- Python 3 (standard library only)
- The `claude` CLI on your `PATH`
- [`fzf`](https://github.com/junegunn/fzf), only needed for `cctree pick`

## Installation

```sh
chmod +x cctree
ln -s "$PWD/cctree" ~/.local/bin/cctree   # or any other directory on your PATH
```

## Usage

Run `cctree` from inside a project directory. It reads the sessions from `~/.claude/projects/<project>/`.

```sh
cctree                  # tree of branched conversations in this project
cctree -a               # also list single-line sessions that never branched
cctree show <id>        # list the numbered prompts along a session's path
```

`<id>` can be any unique prefix of a session id, such as the 8 characters shown in the tree.

### Example output

```
» refactor the parser (+3 more) … » add tests for edge cases
  ◆ 3f2a9c1e parser-tests 09-27 14:02 · 6 prompts
  │ » try a recursive-descent approach instead
  │   ◆ a81d04b7 parser-rd 09-27 15:40 · 5 prompts
```

Each `»` line is a prompt. When a stretch of prompts has no branches, it is collapsed into `(+N more)`. Each `◆` line is a session that ends at that point.

### Resuming and forking

```sh
cctree r <id> [name]    # claude --resume <id>
cctree f <id> [name]    # claude --resume <id> --fork-session [-n name]
```

### Forking from an earlier point

```sh
cctree f <id> --at N "new prompt" [name]
```

This keeps only the first `N` turns of `<id>` (use `cctree show` to see the numbers), sends `"new prompt"` as the next question in print mode, and then opens the new fork interactively. The original session is left unchanged.

### Redoing an earlier prompt

```sh
cctree e <id> K [name]
```

This works like rewinding, except that it happens in a copy. It writes a new session that contains only the turns before prompt `K`, then opens it with prompt `K` already typed into the input box, ready to edit. No model call is made, and the original session is kept as it is.

### Interactive picker

```sh
cctree pick
```

This opens an `fzf` list of all sessions with a preview of their prompts:

| Key      | Action                                              |
|----------|-----------------------------------------------------|
| `enter`  | resume the session                                  |
| `ctrl-f` | fork the session                                    |
| `ctrl-t` | pick an earlier prompt to redo (same as `cctree e`) |

## Notes

- Sessions are opened in the working directory they were recorded in, if that directory still exists.
- Output is uncolored when it is piped. Set `CCTREE_COLOR=1` to force colors.
- `cctree e` writes a new `.jsonl` file directly into `~/.claude/projects/<project>/`.
