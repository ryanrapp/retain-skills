---
name: retain-curate-work
description: Goes through this repository's Working build packets four at a time, recommends Archive, Drop, Finish or Keep for each, and does what the user picks. Use when the user runs /retain:curate-work or asks to go through their Work list here. Do not use to build a packet or for tasks or threads.
compatibility: Requires the retain CLI and a checkout of the repository. The Retain MCP connector alone cannot curate.
---

# Curate this repository's Work

Retain's Work list shows a packet as Working while some of its tasks are ticked and some are not. Many of those are finished: the code shipped, but the last tasks were never ticked, or an imported OpenSpec change was never archived in the repository. This skill does the legwork of a backlog review, one packet at a time, and leaves every decision to the user. `references/cli.md` gives the exact commands.

## What this skill may do

The user asked for this curation, so changing their packets is the point of it, and other skills' rule against offering to clean up Retain's lists does not stop it. Those rules are about tasks and threads, and this skill never touches either. It changes a packet only after the user has picked that packet's decision in this run, and never more than the four packets it showed. There is no "all" choice, and none may be offered.

## Before anything

- There must be a checkout here: run `git rev-parse --show-toplevel`. Outside one, say curating needs a checkout of the repository and stop.
- Run `retain packet list --state building --state done --limit 50` in the checkout. `retain packet list` answering `unrecognized subcommand` means the CLI is too old: say so with how to update it (`references/cli.md`) and stop.

## 1. Say what is covered

Open with one or two lines naming the repository (`repo` from the list) and the counts from `totals`: how many of the user's Working packets (`building`) are in this repository (`here`) and how many in all (`all`), and how many built packets (`done`) are waiting to be archived here. Say that packets of other repositories are not covered and are curated from a checkout of theirs. For example: "Curating github.com/acme/app: 12 of your 65 Working packets are here, and 2 built ones wait to be archived. The other 53 belong to other repositories."

## 2. Take the next four

From the listed packets, oldest change first, skip the ones the user kept (below) and take the first four. When none are left, say the repository's Working list is curated and stop.

## 3. Read each one

For each of the four, gather the evidence, reading only:

- **Progress:** `done` of `total` from the list. `retain packet status <id>` gives the local copy's folder (`local.dir`) when it was pulled on this computer.
- **Tasks still open:** read `tasks.md` from the local copy, or for an `openspec/1` packet from `openspec/changes/<name>/tasks.md` in the checkout. For each unchecked task, look in the code for whether it was done: a short search for the names it mentions, not a full review. When neither copy is here, say the tasks were not read.
- **Pull requests:** the list's `pullRequests` with their state. `gh pr view <url> --json state,mergedAt` reads one's current state when `gh` is available.
- **Someone building it:** `agentActiveAt` within the last day.

## 4. Recommend

One recommendation per packet, in this order:

- An agent reported within the last day, or a pull request is open → **Keep**: it is in flight.
- Every task is done, or the unchecked ones are in the code, or its pull request merged and what is left is trivial → **Archive**: it is built.
- Real tasks are left and the work still makes sense → **Finish**.
- No real progress, superseded by other work, or no longer wanted → **Drop**.

The reason is one line naming the evidence: "PR #412 merged; tasks 4.2 and 4.3 are in `export.ts`", or "Nothing ticked since October; next-work replaces it".

## 5. Ask once

Show the four as a short numbered list: name, title, `done/total`, pull requests, and the recommendation with its reason. Then ask:

- In Claude Code, one AskUserQuestion call with one question per packet. Each offers Archive, Drop, Finish and Keep, with the recommendation first and marked (Recommended).
- Elsewhere, ask for a reply such as "1 archive, 2 keep, 3 finish, 4 drop", and read back what you understood before acting.

A packet the user did not answer for is Keep for this run, and is not recorded as kept.

## 6. Carry it out

Each decision on its own. Report what each command returned, and when one fails, say so and go on with the others.

- **Archive**
  - A `retain-packet/1` or `retain-packet/2` packet: `retain packet archive <id>`, with `--force` when tasks are unchecked, which records them as open in the snapshot. It needs the packet pulled on this computer. If it is not, `retain packet pull <id>` first, then archive.
  - An `openspec/1` packet: `openspec archive <name> --yes` in the checkout, then `retain sync`, which archives the packet in Retain. Say the repository now has the archived change to commit. Never commit it for the user.
- **Drop:** `retain packet drop <id> --reason "<the reason>"`. For an imported change it deletes `openspec/changes/<name>` too, and refuses while that folder has uncommitted changes; show the files it names and leave it. Say the deletion is the user's to commit. Say that `retain packet restore <id>` brings a dropped packet back.
- **Finish:** change nothing. List `/retain:apply <name>` for each, to run one at a time, each in its own session or worktree.
- **Keep:** change nothing in Retain. Record it as kept (below).

Then say what is next: "Run /retain:curate-work again for the next four (8 left)".

## Kept packets

Kept packets are remembered on this computer only, in `curation.json` in this checkout's folder of Retain's device store: `~/.retain/projects/<checkout path with every / and . replaced by ->/curation.json` (under `RETAIN_CONFIG_DIR` instead when it is set). It is a JSON object keyed by packet id:

```json
{ "pk_4jcvr9rj": { "decision": "keep", "at": "2026-11-21T14:00:00Z", "updatedAt": "2026-11-20T09:12:00.000Z", "done": 6, "total": 9 } }
```

- `updatedAt`, `done` and `total` are the packet's, as the list showed them when kept.
- A kept packet is skipped while all three are unchanged and `at` is less than 30 days ago. Otherwise it is shown again, and its entry is replaced when the user decides again.
- Only this skill writes the file, and it never leaves the computer. A missing or unreadable file means nothing is kept.

## Honesty

- Report what the tools returned. Never say a packet is archived or dropped unless the command said so.
- The recommendation is yours and the decision is the user's. Never act on a recommendation the user did not pick.
- Evidence you did not read is not evidence: say when a packet's tasks or pull requests could not be checked.
