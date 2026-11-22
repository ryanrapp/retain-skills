# retain-design: retain CLI

Writing a Retain design. The design is written with the retain CLI: the parts are written in the packet's folder under the packet root, validated, and the packet is created in Retain once both pass. Retain is read with the MCP tools when the connector is available, otherwise with the CLI.

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
| Find passages across meetings, memories and threads, earlier technical designs by the words or meaning of their body (scope designs), or threads, tasks or other documents Retain wrote with the user (scope documents) | `retain search` |
| Read one item by the reference a search or list returned (thread:, memory:, task:, action:, asset:) | `retain fetch` |
| Read where packets go for this checkout, and its repository folder | `retain packet root` |
| Read the instruction and template for the proposal or the design | `retain packet instructions` |
| Validate the packet folder; every issue has a rule, a message and a fix | `retain packet validate` |
| Check whether this repository already has a packet of the name | `retain packet status` |
| List this repository's packets the change may build on | `retain packet list` |
| See or change the packets a packet builds on | `retain packet depends` |
| Create the packet in Retain from the folder, once the proposal and the design are valid | `retain packet create` |

## What each command is for

- `retain search`: Search the knowledge base. Flags: `--text`, `--text-file`, `--limit`, `--scope`, `--kind`, `--state`, `--since`, `--cursor`.
- `retain fetch`: Read one item in full by the reference search or task list returned (thread:, memory:, task:, action:, asset:). Flags: `--detail`, `--at`, `--around`, `--window`, `--offset`.
- `retain packet root`: Where packets are written for the current checkout and where that came from: --dir, RETAIN_PACKETS_ROOT, packets.root for this repository, packets.root, or the default under ~/.local/share/retain/packets. Flags: `--dir`.
- `retain packet instructions`: The instruction, template, dependencies and unlocks for one packet part (proposal, design, requirements or tasks), read against the local copy when it has been pulled. Flags: `--packet`, `--dir`.
- `retain packet validate`: Validate a packet's parts: every issue has a rule, a message and a fix showing the expected shape. Takes a packet id (its local copy when pulled) or a packet folder. Flags: `--dir`.
- `retain packet status`: A packet's state, task progress, validation and where its local copy is. Flags: `--dir`.
- `retain packet list`: This repository's packets in the states asked for, oldest work change first, each with its task progress, pull requests and when an agent last built it; with the person's counts per state across every repository and in this one. Flags: `--state`, `--repo`, `--limit`, `--cursor`.
- `retain packet depends`: The packets of this repository a packet builds on, each with whether it is far enough along to build on (a pull request open, draft or merged, or done or archived), and the packets that build on it. --add and --remove change them; itself, unknown packets and cycles are refused. Flags: `--add`, `--remove`, `--dir`.
- `retain packet create`: Create a packet from a design written here: a folder under the packet root holding a valid proposal.md and design.md. Retain validates both and writes nothing unless both pass; the folder then is a pulled packet. Flags: `--thread`, `--depends-on`, `--title`, `--dir`.
