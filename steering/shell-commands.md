# Writing shell commands that match the permission rules

Kiro decides whether a shell command needs approval by matching its literal
text against `~/.kiro/settings/permissions.yaml`. It splits a command at `;`,
`&&`, `||` and `|`, checks each part on its own, and never expands variables or
shell functions. A command that does the right thing can still prompt if its
text doesn't match an allow rule.

## Running tools

Run a tool by its interpreter path, or through a variable that holds that path.
Assigning the variable is allowed for these names: `T`, `S`, `PY`, `TOOL`,
`SKILL`.

```
tools/my-tool/.venv/bin/python tools/my-tool/scripts/run.py status --env dev
```

```
T=tools/my-tool; $T/.venv/bin/python $T/scripts/run.py get item "name" --env dev
```

```
PY="tools/my-tool/.venv/bin/python"; $PY tools/my-tool/scripts/run.py show 1234 --env dev
```

```
SKILL=~/.kiro/skills/my-skill; $SKILL/.venv/bin/python $SKILL/scripts/run.py get ITEM-1234
```

Do **not** define a shell function and call it without a `$`:

```
run() { $T/.venv/bin/python $T/scripts/run.py "$@"; }; run status 1234
```

Both `run() { ... }` and `run status ...` miss every allow rule, so each part
prompts.

Keep a variable assignment on its own, separated by `;` or `&&`. `T=x cmd`
(assignment as a prefix to a command) and values containing `$(...)` or
backticks prompt.

Python is allowed from a repo's own `.venv/bin/`, `~/Workspace/...`, pyenv,
`~/.kiro/skills/*/.venv/bin/`, and `python`/`python3` on PATH.

## Temporary files

Put scratch files under `/tmp` with a literal `/tmp/` prefix, e.g.
`rm -rf /tmp/my-review` or `cp "$f" "/tmp/$(basename $f).pre"`. A bare
`rm -rf "$D"` prompts even when `$D` came from `mktemp`, because the rule can't
see what `$D` holds. Use `/tmp/...` paths, not `..`, and up to 8 `rm` targets
per command.

## Command shape

Set the working directory with the shell tool's `cwd` parameter. Don't start a
command with `cd <dir> &&`.

Keep each command flat: simple commands joined by `;`, `&&` or `|`. These
constructs miss every allow rule and prompt:

- `for`/`while` loops (`for x in ...; do ...; done`). Run one command per item,
  or put the loop in a script under `/tmp/`.
- Heredocs (`python - <<'EOF' ... EOF`, `cat > f <<EOF`). The splitter cuts
  the body at any `;` or `|` inside it.
- Inline code longer than a line or containing `;` or `|`, such as
  `python -c "a; b"`.
- Lines continued with a trailing `\`.

For multi-line Python, write it to a file under `/tmp/` with the file-write
tool, then run it:

```
uv run python /tmp/check_script.py
```

```
.venv/bin/python /tmp/check_script.py
```

## One command per location

`*` in an allow rule also matches spaces, so `rm -rf /tmp/a /home/user/x`
matches `rm -rf /tmp/*` and runs with no prompt. Kiro only splits at `;`, `&&`,
`||` and `|`, never between arguments.

Always split such a command so each part names paths from one location:

```
rm -rf /tmp/a /home/user/x
```

becomes

```
rm -rf /tmp/a; rm -rf /home/user/x
```

- Applies to `rm`, `rmdir`, `unlink`, `mv`, `chmod` and `tee`.
- Several `/tmp` targets may share one command (`rm -rf /tmp/a /tmp/b`).
- Never put a `/tmp` path and a non-`/tmp` path (`/home/...`, `~`, `$HOME`, a
  relative path, a glob) in the same command. Run the non-`/tmp` part as its
  own command so it gets its own prompt or deny.

## Environment variables

Don't put environment variables at the front of a command
(`env MY_VAR=ci ...`, `MYPYPATH=src:tests uv run ...`,
`APP_CONFIG=... uv run ...`). Each new name or value misses every allow rule.

Write the variables to a file under `/tmp/` with the file-write tool, one
`NAME=value` per line, then pass it to `uv run --env-file`:

```
MY_VAR=ci
MYPYPATH=src:tests
```

```
uv run --env-file /tmp/kiro-test.env pytest -q tests/unit
```

Name the file `/tmp/kiro-<purpose>.env`. Never name it `.env` or `.env.*`.
Never put secrets in it. Never set `PATH` this way; use `PYTHONPATH` or
`MYPYPATH` for import paths.

## Prompts that are intentional

These always ask. Don't restructure a command to get past them:

- Force push, `git reset --hard`, `git clean`, `git branch -D`
