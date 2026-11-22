# retain-apply: retain CLI

Building a Retain build packet. Apply needs the retain CLI: the packet is pulled into the packet root, its requirements and tasks are written and validated, pushed back, and then built.

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
| Write the packet's files into its folder under the packet root and link it to its thread | `retain packet pull` |
| Read the instruction, template and dependencies for one part | `retain packet instructions` |
| Validate the packet folder; every issue has a rule, a message and a fix | `retain packet validate` |
| Send the parts written here back to Retain | `retain packet push` |
| Check the packets it builds on before building: whether each has a pull request to stack onto, and the branch the build starts from | `retain packet deps` |
| Claim the packet for this build so no other agent builds it at the same time | `retain packet claim` |
| End this build's claim when stopping before the last task | `retain packet release` |
| Tell Retain about this build: its pull request, and the agent's session when the CLI cannot see it | `retain packet session` |
| Read the packet's state, progress and local folder | `retain packet status` |
| Freeze the finished packet as a snapshot of what was built | `retain packet archive` |
| Send task progress to Retain | `retain sync` |

## What each command is for

- `retain packet pull`: Write a build packet's files into its folder under the packet root (outside the repo unless configured), link it to its thread, and report what is missing and how far the checkout has moved since the design. Flags: `--dir`, `--force`, `--exclude`.
- `retain packet instructions`: The instruction, template, dependencies and unlocks for one packet part (proposal, design, requirements or tasks), read against the local copy when it has been pulled. Flags: `--packet`, `--dir`.
- `retain packet validate`: Validate a packet's parts: every issue has a rule, a message and a fix showing the expected shape. Takes a packet id (its local copy when pulled) or a packet folder. Flags: `--dir`.
- `retain packet push`: Send the parts changed locally back to Retain. Each is validated; a part Retain changed since the pull is refused and both versions are returned. Flags: `--dir`.
- `retain packet deps`: Check a packet's dependencies before building it: reads each one's pull requests not merged yet with the GitHub CLI (gh), then says whether each is ready (a pull request open, draft or merged, or done or archived; tests and archiving are not required) and what the build starts from: the default branch, the one dependency branch to stack onto, or several to choose from. Without gh it answers from what Retain has stored, with refreshed false. Flags: `--dir`.
- `retain packet claim`: Claim a packet for this checkout's build, so no other agent builds it at the same time. Succeeds only when no live claim exists or the claim is this checkout's own; otherwise refuses with claimed and who holds it. A claim lasts 30 minutes and this build's activity renews it (push, sync, packet session, packet heartbeat). It ends when the last task is pushed done, on archive or drop, on packet release, or when it lapses. --take-over takes a lapsed claim of another build: check its branch for commits it never pushed first. Flags: `--take-over`, `--dir`.
- `retain packet release`: End this checkout's claim on a packet, so another build may take it. A claim this checkout does not hold is left as it is. Flags: `--dir`.
- `retain packet session`: Link this checkout's build of a packet to its Build tab: the worktree, branch and the agent's session, and with --pr the build's pull request. Nothing is linked unless you run this: pull, push and sync only keep a build already linked here up to date, and never link a pull request. --remove unlinks this checkout's build; --remove-session unlinks any build by the id the packet lists. Flags: `--pr`, `--remove`, `--remove-session`, `--session-url`, `--session-label`, `--agent`, `--dir`.
- `retain packet status`: A packet's state, task progress, validation and where its local copy is. Flags: `--dir`.
- `retain packet archive`: Freeze a finished packet as a snapshot of what was built: records the commit range and changed files, moves the folder under archive/ and makes it read-only in Retain. Never merges into long-lived specs. A packet not pulled here, such as an imported OpenSpec change, is archived from Retain's copy of its tasks: Retain holds the archive state, whatever the repository's openspec folders say. Flags: `--dir`, `--force`.
- `retain sync`: Refresh the local work you linked to threads, send today's progress, and return Retain's digest for today.
