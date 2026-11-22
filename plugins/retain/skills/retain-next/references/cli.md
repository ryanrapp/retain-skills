# retain-next: retain CLI

Picking the next build. It needs the retain CLI and a checkout: Retain ranks this repository's free packets, the user picks one, it is claimed in its own worktree and built with retain-apply.

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
| Rank this repository's free buildable packets, or claim the chosen one for this worktree | `retain packet next` |
| Take over a lapsed claim, once the user agrees | `retain packet claim` |
| Record what the claim could not see about this build (its pull request, the agent's session) | `retain packet session` |
| Read the thread a packet advances | `retain work show` |

## What each command is for

- `retain packet next`: Rank this repository's packets that are ready to build (handed-off, planned or building) and that no other build has claimed, and give the top pick with its reasons and the next two. Changes nothing. --claim <name> claims that packet for the checkout it runs in, as packet claim does; if another build took it meanwhile it refuses with claimed and offers the next free one as replacement. Flags: `--claim`.
- `retain packet claim`: Claim a packet for this checkout's build, so no other agent builds it at the same time. Succeeds only when no live claim exists or the claim is this checkout's own; otherwise refuses with claimed and who holds it. A claim lasts 30 minutes and this build's activity renews it (push, sync, packet session, packet heartbeat). It ends when the last task is pushed done, on archive or drop, on packet release, or when it lapses. --take-over takes a lapsed claim of another build: check its branch for commits it never pushed first. Flags: `--take-over`, `--dir`.
- `retain packet session`: Link this checkout's build of a packet to its Hand-off: the worktree, branch and the agent's session, and with --pr the build's pull request. Nothing is linked unless you run this: pull, push and sync only keep a build already linked here up to date, and never link a pull request. --remove unlinks this checkout's build; --remove-session unlinks any build by the id the packet lists. Flags: `--pr`, `--remove`, `--remove-session`, `--session-url`, `--session-label`, `--agent`, `--dir`.
- `retain work show`: What Retain knows about the linked work in this folder: its thread, what is owed, who is waiting, and what changed since you last looked.
