# retain-sync: retain CLI

Syncing with Retain. The sync itself needs the retain CLI; with only the Retain connector, read the digest.

The `retain` CLI is installed by the Retain desktop app at `~/.local/bin/retain`. It may not be on `PATH`, so call `$HOME/.local/bin/retain` when `retain` is not found.

- Run `retain schema` for the exact commands and flags, and `retain schema <command...>` for one command. Prefer it over guessing. `--help` is for humans.
- Every command prints a JSON envelope on stdout and diagnostics on stderr. Check `"ok"` before using the result. List commands paginate: pass `--limit N`, or `--all --max-items N` when you need more than one page.
- Run `retain auth status` first. If the user is not signed in, run `retain auth login`, which opens a browser, and ask them to finish signing in. Never ask the user to paste a token into a command line.
- Mutating commands accept `--request-id <uuid>`. Reuse the same id when retrying so the server does not apply the change twice.
- Pass long text with `--text-file <path>` or on stdin (`--text-file -`) instead of the command line.
- A thread update takes `--kind`, `--origin agent`, `--host <your host>` (`claude`, `chatgpt`, `codex` or `other`) and `--authorization <user_requested|user_confirmed>`; add `--actor other` when the user reports someone else's move. The server records the host it observed from your sign-in when it can.

| To | Command |
|---|---|
| Send today's progress for linked work and get the digest back | `retain sync` |
| Read Retain's digest without syncing (a past day with a date) | `retain digest` |
| Link a suggested item to its thread, or dismiss the suggestion | `retain work link` |
| Record finished work the user confirmed as an outcome on its thread | `retain thread update add` |
| Read a thread back after an update | `retain fetch` |

## What each command is for

- `retain sync`: Refresh the local work you linked to threads, send today's progress, and return Retain's digest for today.
- `retain digest`: Read Retain's daily digest (today's, or another day's) without syncing. Flags: `--date`.
- `retain work link`: Link the local work in this folder (an OpenSpec change or a checklist) to the thread it advances. Without --to, lists the work here and the best-matching threads. Flags: `--to`, `--action`, `--remove`, `--dismiss`, `--origin`, `--host`, `--authorization`.
- `retain thread update add`: Add a small update the user confirmed to a thread (append-only; never a draft passed off as done). Flags: `--text`, `--text-file`, `--request`, `--request-file`, `--kind`, `--actor`, `--artifact-url`, `--occurred-at`, `--origin`, `--host`, `--authorization`.
- `retain fetch`: Read one item in full by the reference search or task list returned (thread:, memory:, task:, action:). Flags: `--detail`, `--at`, `--around`, `--window`.
