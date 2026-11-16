# retain-explore: retain CLI

Exploring a change. Read Retain with the MCP tools when the connector is available, otherwise with the retain CLI. Every operation here reads; explore writes nothing to Retain.

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
| Find passages across meetings, memories and threads (scope memories, the default); threads or tasks; or documents Retain wrote with the user, such as earlier designs, by the words or meaning of their body, filtered by kind, state and since (scope documents; scope designs for design docs only) | `retain search` |
| Read one item by the reference a search or list returned (thread:, memory:, task:, action:, asset:) | `retain fetch` |
| See what is going on in the user's threads | `retain thread list` |
| See what the user has to do: open tasks and the actions suggested on their threads | `retain task list` |
| Read one thread's record | `retain thread get` |
| See what Retain knows about the linked work in this folder before exploring it | `retain work show` |

## What each command is for

- `retain search`: Search the knowledge base. Flags: `--text`, `--text-file`, `--limit`, `--scope`, `--kind`, `--state`, `--since`, `--cursor`.
- `retain fetch`: Read one item in full by the reference search or task list returned (thread:, memory:, task:, action:, asset:). Flags: `--detail`, `--at`, `--around`, `--window`, `--offset`.
- `retain thread list`: List threads with their section, status headline and next step. Flags: `--limit`, `--cursor`, `--all`, `--max-items`, `--section`.
- `retain task list`: List what the user has to do: open tasks and the actions suggested on their threads. Flags: `--limit`, `--cursor`, `--all`, `--max-items`, `--status`, `--priority`.
- `retain thread get`: Read a thread.
- `retain work show`: What Retain knows about the linked work in this folder: its thread, what is owed, who is waiting, and what changed since you last looked.
