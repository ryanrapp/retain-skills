# retain-capture: retain CLI

Saving to Retain. Use the MCP tools when the Retain connector is available, otherwise the retain CLI.

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
| Find an existing item first, so a save updates it instead of duplicating it | `retain search` |
| See the user's open tasks and thread suggestions before adding a task | `retain task list` |
| See which threads need the user's attention (the weekly review) | `retain thread list` |
| Read a thread before reporting on it | `retain fetch` |
| Add a task the user asked for | `retain task create` |
| Change a task, for example mark it done | `retain task update` |
| Record something the user confirmed on a thread (a decision, something done, an outcome) | `retain thread update add` |
| Read a thread's recorded updates | `retain thread update list` |
| Import the user's own notes or a transcript as source material | `retain import` |
| Upload a file for an import to attach | `retain file upload` |
| Link work in progress on the user's computer to the thread it advances | `retain work link` |

## What each command is for

- `retain search`: Search the knowledge base. Flags: `--text`, `--text-file`, `--limit`.
- `retain task list`: List what the user has to do: open tasks and the actions suggested on their threads. Flags: `--limit`, `--cursor`, `--all`, `--max-items`, `--status`, `--priority`.
- `retain thread list`: List threads with their section, status headline and next step. Flags: `--limit`, `--cursor`, `--all`, `--max-items`, `--section`.
- `retain fetch`: Read one item in full by the reference search or task list returned (thread:, memory:, task:, action:). Flags: `--detail`, `--at`, `--around`, `--window`.
- `retain task create`: Create a task. Flags: `--request`, `--request-file`, `--title`, `--description`, `--description-file`, `--priority`, `--due`, `--id`.
- `retain task update`: Patch a task. Flags: `--request`, `--request-file`, `--title`, `--description`, `--description-file`, `--clear-description`, `--status`, `--priority`, `--clear-priority`, `--due`, `--clear-due`, `--if-version`.
- `retain thread update add`: Add a small update the user confirmed to a thread (append-only; never a draft passed off as done). Flags: `--text`, `--text-file`, `--request`, `--request-file`, `--kind`, `--actor`, `--artifact-url`, `--occurred-at`, `--origin`, `--host`, `--authorization`.
- `retain thread update list`: List a thread's updates, newest first. Flags: `--limit`, `--cursor`, `--all`, `--max-items`.
- `retain import`: Import the user's own notes, transcripts or files as source material for Retain to interpret. Flags: `--text`, `--text-file`, `--request`, `--request-file`, `--json`, `--json-file`, `--local-file`, `--attach`, `--title`, `--intent`, `--dry-run`.
- `retain file upload`: Upload a local file or stdin as a durable file. Flags: `--name`, `--content-type`.
- `retain work link`: Link the local work in this folder (an OpenSpec change or a checklist) to the thread it advances. Without --to, lists the work here and the best-matching threads. Flags: `--to`, `--action`, `--remove`, `--dismiss`, `--origin`, `--host`, `--authorization`.
