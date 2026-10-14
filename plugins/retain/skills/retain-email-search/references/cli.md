# retain-email-search: retain CLI

Reading the user's email. Use the MCP tools when the Retain connector is available, otherwise the retain CLI. Both work only where email search is enabled.

The `retain` CLI is installed at `~/.local/bin/retain` by Retain's setup for Claude Code or Codex (in the Retain Mac app, Settings > AI assistants) or by the Retain desktop app. It may not be on `PATH`, so call `$HOME/.local/bin/retain` when `retain` is not found.

If it is in neither place, or it answers `unrecognized subcommand` for a command these notes name (an older CLI), say so and tell the user how to get the current one: run Retain's setup for their assistant again, or install it themselves with `curl -fsSL -o install.sh https://releases.retainapp.ai/cli/latest/install.sh` and then `RETAIN_DEST="$HOME/.local/bin" sh install.sh`, which checks the download before installing. Do not run the installer for them unasked, and do not reach for another `retain` binary.

- Run `retain schema` for the exact commands and flags, and `retain schema <command...>` for one command. Prefer it over guessing. `--help` is for humans.
- Every command prints a JSON envelope on stdout and diagnostics on stderr. Check `"ok"` before using the result. List commands paginate: pass `--limit N`, or `--all --max-items N` when you need more than one page.
- Run `retain auth status` first. If the user is not signed in, run `retain auth login`, which opens a browser, and ask them to finish signing in. Never ask the user to paste a token into a command line.
- Mutating commands accept `--request-id <uuid>`. Reuse the same id when retrying so the server does not apply the change twice.
- Pass long text with `--text-file <path>` or on stdin (`--text-file -`) instead of the command line.
- A thread update takes `--kind`, `--origin agent`, `--host <your host>` (`claude`, `chatgpt`, `codex` or `other`) and `--authorization <user_requested|user_confirmed>`; add `--actor other` when the user reports someone else's move. The server records the host it observed from your sign-in when it can.

| To | Command |
|---|---|
| List the connected Gmail accounts and whether each can be searched | `retain email accounts` |
| Search by Gmail syntax (native), by meaning (semantic) or both (hybrid), with hard constraints | `retain email search` |
| Read one email by the ref a search returned | `retain email read` |
| Read the messages around a hit in its Gmail conversation | `retain email conversation` |

## What each command is for

- `retain email accounts`: List the user's connected Gmail accounts and whether each can be searched.
- `retain email search`: Search email by Gmail query (default), by meaning (--mode semantic --semantic-text), or both (--mode hybrid). Returns refs for `retain email read`, never an answer; one page of one query is not the whole mailbox, and a meaning search covers only the part of it Retain has indexed. Flags: `--query`, `--semantic-text`, `--mode`, `--question`, `--account`, `--from`, `--to`, `--after`, `--before`, `--subject-term`, `--include-spam-trash`, `--limit`, `--cursor`.
- `retain email read`: Read one email in full by the ref `retain email search` returned (email:<account>:<message>).
- `retain email conversation`: Read the messages around a search hit in its Gmail conversation, oldest first, each with its own ref. Flags: `--match`, `--cursor`, `--limit`.
