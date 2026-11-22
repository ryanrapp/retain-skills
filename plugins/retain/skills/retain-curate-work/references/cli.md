# retain-curate-work: retain CLI

Curating the Work list for this repository. Curation needs the retain CLI and a checkout: packets are listed, checked against the code and their pull requests, and archived or dropped only as the user picks.

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
| List this repository's Working and built packets, oldest change first, with the counts everywhere and here | `retain packet list` |
| Read a packet's state and where its local copy is | `retain packet status` |
| Pull a packet that must be archived and has no local copy here | `retain packet pull` |
| Archive a built packet as a snapshot of what was built | `retain packet archive` |
| Drop a packet that will not be built | `retain packet drop` |
| Bring a dropped packet back | `retain packet restore` |

## What each command is for

- `retain packet list`: This repository's packets in the states asked for, oldest work change first, each with its task progress, pull requests and when an agent last built it; with the person's counts per state across every repository and in this one. Flags: `--state`, `--repo`, `--limit`, `--cursor`.
- `retain packet status`: A packet's state, task progress, validation and where its local copy is. Flags: `--dir`.
- `retain packet pull`: Write a build packet's files into its folder under the packet root (outside the repo unless configured), link it to its thread, and report what is missing and how far the checkout has moved since the design. Flags: `--dir`, `--force`, `--exclude`.
- `retain packet archive`: Freeze a finished packet as a snapshot of what was built: records the commit range and changed files, moves the folder under archive/ and makes it read-only in Retain. Never merges into long-lived specs. A packet not pulled here, such as an imported OpenSpec change, is archived from Retain's copy of its tasks: Retain holds the archive state, whatever the repository's openspec folders say. Flags: `--dir`, `--force`.
- `retain packet drop`: Drop a packet that will not be built: Retain keeps it as a record, off the live Work list, with the tasks it left open and why. For an imported OpenSpec change it also deletes openspec/changes/<name> here, refusing while that folder has uncommitted changes. retain packet restore undoes it. Flags: `--reason`, `--dir`.
- `retain packet restore`: Restore a dropped packet to the state its files and task progress give it. A packet that is not dropped is left as it is.
