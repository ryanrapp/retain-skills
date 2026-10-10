# retain-search: retain CLI

Reading Retain. Use the MCP tools when the Retain connector is available, otherwise the retain CLI.

The `retain` CLI is installed by the Retain desktop app at `~/.local/bin/retain`. It may not be on `PATH`, so call `$HOME/.local/bin/retain` when `retain` is not found.

- Run `retain schema` for the exact commands and flags, and `retain schema <command...>` for one command. Prefer it over guessing. `--help` is for humans.
- Every command prints a JSON envelope on stdout and diagnostics on stderr. Check `"ok"` before using the result. List commands paginate: pass `--limit N`, or `--all --max-items N` when you need more than one page.
- Run `retain auth status` first. If the user is not signed in, run `retain auth login`, which opens a browser, and ask them to finish signing in. Never ask the user to paste a token into a command line.
- Mutating commands accept `--request-id <uuid>`. Reuse the same id when retrying so the server does not apply the change twice.
- Pass long text with `--text-file <path>` or on stdin (`--text-file -`) instead of the command line.
- A thread update takes `--kind`, `--origin agent`, `--host <your host>` (`claude`, `chatgpt`, `codex` or `other`) and `--authorization <user_requested|user_confirmed>`; add `--actor other` when the user reports someone else's move. The server records the host it observed from your sign-in when it can.

| To | Command |
|---|---|
| Find passages across meetings, memories and threads | `retain search` |
| Read one item by the reference a search or list returned (thread:, memory:, task:, action:) | `retain fetch` |
| See what is going on in the user's threads, Action owed first | `retain thread list` |
| See what the user has to do: open tasks and the actions suggested on their threads | `retain task list` |
| Read one thread's record | `retain thread get` |
| Read one task | `retain task get` |

## What each command is for

- `retain search`: Search the knowledge base. Flags: `--text`, `--text-file`, `--limit`.
- `retain fetch`: Read one item in full by the reference search or task list returned (thread:, memory:, task:, action:). Flags: `--detail`, `--at`, `--around`, `--window`.
- `retain thread list`: List threads with their section, status headline and next step. Flags: `--limit`, `--cursor`, `--all`, `--max-items`, `--section`.
- `retain task list`: List what the user has to do: open tasks and the actions suggested on their threads. Flags: `--limit`, `--cursor`, `--all`, `--max-items`, `--status`, `--priority`.
- `retain thread get`: Read a thread.
- `retain task get`: Get a task.
