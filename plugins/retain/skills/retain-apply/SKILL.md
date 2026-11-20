---
name: retain-apply
description: Builds a named Retain build packet here, including an OpenSpec change that retain sync imported: pulls it, writes missing requirements and tasks, pushes them, builds it task by task. Use when the user runs /retain:apply. Do not use to edit OpenSpec changes in the repository.
compatibility: Requires the retain CLI and a checkout of the packet's repository. The Retain MCP connector alone cannot apply a packet.
---

# Apply a Retain build packet

A build packet is a design Retain wrote with the user: `proposal.md` (why, what changes, what is left out) and `design.md` (the decision and how it fits what exists). You write the other half here, against the real code: `requirements.md` and `tasks.md`. Then you build it. `references/cli.md` gives the exact commands.

An OpenSpec change that `retain sync` imported is a packet too, with format `openspec/1`, named like its folder (`ios-work`). Its parts came from the repository's `openspec/changes/<name>` (the specs joined into `requirements.md`, one section per capability). Retain neither checks them nor writes them: you build from what is there.

The arguments are the packet's name (lowercase words joined by hyphens, as Retain shows it on the packet's Hand-off tab) and, optionally, `--review`. Without a name, ask for it. Older commands may pass an id that starts with `pk_`; it works the same. Use the id `retain packet pull` prints from then on.

## Ground rules

- The CLI is the contract. Take every instruction, template and check from `retain packet instructions` and `retain packet validate`; never from memory of another spec format. Pull reports the packet's `format`: only a `retain-packet/1` packet has instructions and checks.
- The packet folder is the working copy, even for an imported OpenSpec change. Never edit the repository's `openspec/changes` folder for it: that copy is shared, and the packet is the user's.
- Packet files live in the folder `retain packet pull` prints as `dir`. It may be outside the repository (the default is under `~/.local/share/retain/packets`). Code changes go in the checkout; packet changes go in `dir`. Never copy packet files into the repository.
- If writing in `dir` is refused, the assistant was started without access to it. Stop and tell the user the flag to add when starting it: `--add-dir <dir's packet root>` for Claude Code or Codex.
- What the user decided in Retain is the direction. Do not redesign it; if the code makes the design wrong, say so and ask before departing from it.

## Steps

1. **Pull.** Run `retain packet pull <name>` in the repository. It writes the files, links the packet to its thread, and prints `dir`, `state`, `format`, `partsMissing` and `commitsSinceDesign`.
   - An `ambiguous_packet` refusal means several packets have that name and none is this repository's. Ask the user which one, listing each candidate's title, repository, state and last change from the refusal's `candidates`: in Claude Code with the AskUserQuestion tool, elsewhere as a plain question. Then pull it by its `id` and use that id from then on.
   - A refusal for the wrong repository names the expected one: tell the user and stop.
   - A refusal because local files changed: show which, and ask before passing `--force`.
   - `state` drafting: Retain is still writing it; stop and say so.
   - `state` archived: it is a read-only record; read it if asked, build nothing.
2. **Check the design against the code.** Read `proposal.md` and `design.md` from `dir` (an imported change may have no design). When `commitsSinceDesign` is above zero and the design says what exists (a "What exists" or "Context" section), check those claims against the code now and list any that no longer hold. When it has open questions (an "Open questions" section), ask the user about each before building, unless it says None.
3. **Write the missing parts.** For a packet whose `format` is not `retain-packet/1`, write nothing: `partsMissing` lists only `tasks` when there are none, and then stop and say the change has no tasks to build. Otherwise, in order (requirements, then tasks), for each part in `partsMissing`:
   - Run `retain packet instructions <part> --packet <name>`. Follow its `instruction`, fill its `template`, and read the files in its `dependencies` first.
   - Write the part to `dir/<outputPath>`.
   - Run `retain packet validate <name>`. For each issue at error level, fix it as its `message` says, using its `fix` as the shape expected, and validate again. Warnings may stay.
   - At most three rounds per part. If errors remain after the third, stop before building and show the user the remaining issues.
4. **Push.** Run `retain packet push <name>`.
   - A `conflicts` entry means someone edited that part in Retain since the pull: show the user both versions and ask which to keep. Never overwrite Retain's copy without an answer.
   - An `invalid` entry has its issues in `validation`: fix them and push again.
5. **Review stop.** With `--review`, stop here: say the requirements and tasks are in Retain for review, and that running apply again builds it.
6. **Build.** Build on a branch of its own, in a worktree when the repository's conventions (its `AGENTS.md` or `CLAUDE.md`) or the user ask for one; never straight on the default branch. Then work through `tasks.md` in order. Mark each task done (`- [ ]` to `- [x]`) in `dir/tasks.md` as soon as it is finished and verified, not before. Run the tests the requirements' scenarios call for. Stop and ask when a task is unclear, the design does not fit the code, or something fails that you cannot fix. When you finish each group of tasks, and when you stop, run `retain packet push <name>` so the person sees the ticked checklist in Retain, then `retain sync`. A push refused for a conflict means the part changed in Retain: show both versions and ask, as in step 4.
   - Retain links nothing to the packet by itself. Once you are in the worktree and on the branch you build it in, run `retain packet session <name>` there: it links this build (its folder, branch and your session) to the packet's Hand-off, and push and sync keep it up to date from then on. Run it only from this packet's build, never from another change's checkout.
   - When the build is ready for review, open a pull request for its branch as the repository's conventions say (ask when they say nothing), then link it with `retain packet session <name> --pr <url>`. Link only pull requests you opened for this packet.
   - The CLI finds a Claude Code session's link by itself. In Codex, run `retain packet session <name> --agent codex --session-label "codex resume <session id>"` once, when you know your session id.
   - If the Hand-off shows a pull request or build that is not this packet's, unlink it: `retain packet prs <name> --remove <url>` for a pull request, `retain packet session <name> --remove-session <id>` for a build (its id is in `retain packet status <name>`), or `retain packet session <name> --remove` for this checkout's.
7. **Archive.** When every task is checked, offer to archive. On yes, run `retain packet archive <name>`: it records the commit range and the files the build changed, and freezes the packet as a record of what was built. If the user wants to archive with tasks still open, it needs `--force`, which records them in the snapshot; ask first.

## Reporting

- Say which step you are on and what the tools returned. A part that failed validation failed; show its issues.
- Report progress as the tasks file says it is. Never say a task is done that you did not finish and check.
- Archiving is the user's call. It is a snapshot of this build, not a spec anyone keeps up to date.
