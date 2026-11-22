---
name: retain-next
description: Asks Retain which packet here to build next, and after the user picks one claims it in a new worktree and builds it with retain-apply. Use when the user runs /retain:next or asks to work on the next thing. Do not use for a packet the user named (retain-apply).
compatibility: Requires the retain CLI and a checkout of the repository. The Retain MCP connector alone cannot pick or build.
---

# Build the next packet

Retain knows what each of this repository's build packets is waiting on: its thread, its progress, its pull requests, how long it has waited. So Retain picks, the user confirms, and you build. Several agents may be running at once: a packet another build has claimed is never offered, and nothing is claimed until the user says yes. `references/cli.md` gives the exact commands.

## Before anything

- There must be a checkout here: run `git rev-parse --show-toplevel`. Outside one, say picking the next packet needs a checkout of the repository and stop.
- Run `retain packet next` in the checkout. `unrecognized subcommand` means the CLI is too old: say so with how to update it (`references/cli.md`) and stop.

## 1. Say what is covered

One line from `repo` and `totals`: this repository only, its packets ready to build (handed-off, planned or building) that nobody holds. For example: "Looking at github.com/acme/app's packets ready to build: 7 free, 2 being built by other agents." Packets of other repositories are not covered.

## 2. Nothing to build

When `pick` is null, say why: none is ready to build, or every one is claimed by another build (`claimedByOthers`). When `readyToCurate` is above zero, say that many look finished and suggest `/retain:curate-work`. Then stop.

## 3. Ask

Show the pick and each alternative: its name, title, state, `done/total` and its `reasons`, as Retain gave them, and its thread's subject when it has one. Then ask which to build:

- In Claude Code, one AskUserQuestion call with the pick (marked Recommended), each alternative, and None.
- Elsewhere, ask in plain text and read back the one you understood.

None, or no answer, ends the run: claim nothing and change nothing.

## 4. Make its worktree, then claim it from there

The claim is taken from inside the worktree, so it names the folder and branch the build runs in. It belongs to this agent session (where the CLI cannot tell the session, as in Codex, to that worktree).

- Make a worktree beside the checkout, `../<checkout folder>-<name>`, on a new branch `<name>` from the default branch: `git fetch origin` then `git worktree add -b <name> <path> origin/<default branch>`. When that branch or folder already exists, ask the user whether to build there, or stop. When the repository's conventions (its `AGENTS.md` or `CLAUDE.md`) say how worktrees are set up (for example `scripts/worktree-setup.sh`), follow them. When the user declines a worktree, build in place, on a branch of its own.
- In that worktree, run `retain packet next --claim <name>`.
  - It claims the packet for this worktree. Run every command of the build in that folder from then on.
  - `claimed`: another build took it meanwhile. Remove the worktree you just made (`git worktree remove <path>`, then `git branch -D <name>`), say who holds it from `holder`, and offer the `replacement` it returned the same way as step 3. When there is none, stop.
  - `claim_lapsed`: another build held it and its claim ran out. If the holder's `branch` exists here, show its commits no remote has (`git log <branch> --not --remotes --oneline`), then ask the user whether to take it over. Only on yes, run `retain packet claim <name> --take-over` in the worktree; on no, remove the worktree and stop.
  - `not_buildable`: it is no longer ready to build; remove the worktree and run `retain packet next` again.

## 5. Build it

Hand over to `retain-apply` for `<name>` in the worktree: in Claude Code, load the `retain-apply` skill with the Skill tool and its argument `<name>`; elsewhere, follow `retain-apply` by name. Apply pulls the packet, writes what is missing and builds it. Its own claim step finds the claim this session already holds and renews it.

## Honesty

- The pick is Retain's and the choice is the user's. Never claim, make a worktree or build before the user picked one.
- Show the reasons Retain returned; never invent a reason or a ranking of your own.
- Report what each command returned. Never say a packet is claimed unless the command said so.
