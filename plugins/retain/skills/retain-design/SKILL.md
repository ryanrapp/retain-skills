---
name: retain-design
description: Writes the design for a change in this repository as a Retain build packet (proposal and design), from the conversation, the code and the user's Retain, in the direction the user confirms; validates both, then creates the packet for /retain:apply. Use when the user runs /retain:design or accepts the design offer at the end of /retain:explore. Do not use to think a change through (retain-explore), to build a packet (retain-apply), or for other documents.
compatibility: Requires a checkout of the repository the design is for and the retain CLI (with `retain packet create`). The Retain MCP connector alone cannot write a design.
---

# Write a Retain design

A Retain design is the first half of a build packet: `proposal.md` (why, what changes, what is left out) and `design.md` (the decision and how it fits what exists). You write both here, against the real code and what the user's Retain knows, in the direction the user chose. Retain receives nothing until both are valid. The builder writes the other half (requirements and tasks) later with `/retain:apply`. `references/cli.md` gives the exact commands.

This may follow `/retain:explore` in the same conversation, or be the first step. Either way the conversation is the source: what the user said outranks anything you infer.

## Ground rules

- The CLI is the contract. Take every instruction, template and check from `retain packet instructions` and `retain packet validate`; never from memory of another format. The packet is not an OpenSpec change.
- The direction is the user's. Never pick one for them, and never let an older preference found in Retain override what they confirmed here.
- Write only the design half. No requirements, no tasks, no code changes.
- Nothing from this conversation is saved to Retain except the packet: no notes, no thread updates, no tasks marked done.

## Steps

1. **Check what is needed.** There must be a checkout here and the `retain` CLI must have `retain packet create` (run `retain packet create --help`). If either is missing, say which, and how to install or update the CLI, and stop.

2. **Gather what the design needs.** If the conversation has not already done so, read the code the change touches and what Retain knows about it, as the retain-explore skill does: search narrowly, read the source before relying on a hit, treat documents as drafts and memories as the record. Look for an earlier design of the same area with scope designs and two or three distinctive words, trying a few wordings, and name the queries when nothing turns up. Ask about anything you cannot settle from the code or the sources, one or two questions at a time.

3. **Confirm the direction.** If the conversation has not settled one, ask the user which way to go. You may lay out the options you found, with their tradeoffs, but do not choose. Once there is a direction, state it back in one sentence and ask the user to confirm it. Write nothing until they do. That sentence opens the design's Decision section.

4. **Confirm the name and the Thread.**
   - Propose a packet name: lowercase words joined by hyphens, short, naming the change. Ask the user to confirm or change it.
   - Run `retain packet status <name>` in the checkout. A packet of that name for this repository means the name is taken: say so and ask for another. A not-found answer means it is free.
   - If the conversation drew on a Thread this change advances, ask whether to link the design to it. When several could fit, name them and ask which. With no Thread, or a no, the packet has none. Never pick one silently.

5. **Find the folder.** Run `retain packet root --json`. The packet's folder is `<root>/<repoSlug>/<name>` when it gives a `repoSlug`, otherwise `<root>/<name>`. Write the parts there, never in the repository. If writing there is refused, the assistant was started without access to it: stop and tell the user the flag to add when starting it, `--add-dir <root>` for Claude Code or Codex.

6. **Write the proposal, then the design.** For each part in that order:
   - Run `retain packet instructions <part>`. Follow its `instruction` and fill its `template`. For the design, read the proposal you wrote first.
   - Write the part to `<folder>/<outputPath>`.
   - Run `retain packet validate <folder>`. For each issue at error level, fix it as its `message` says, using its `fix` as the shape expected, and validate again. Warnings may stay.
   - At most three rounds per part. If errors remain after the third, stop, show the user the remaining issues, and create nothing.

   What the design cites: in What exists, cite the code you read in this session by file path, and Retain sources by their title and date. Mark anything you did not read here as unverified. History the user's direction departs from goes in as context, not as an objection.

7. **Create the packet.** Run `retain packet create <folder>`, adding `--thread <ref>` only when the user said yes to a Thread, and `--title <title>` when the user named one.
   - An `invalid_request` with issues means a part failed Retain's check: fix it as in step 6 and create again.
   - A `name_taken` refusal means another packet of this repository has the name since you checked: ask for a new name, rename the folder, and create again.
   - Any other failure: say what the command returned and stop.

8. **Report and offer apply.** Say the packet's name, state and folder exactly as `create` returned them, and that it is in Retain for the user to read. Offer `/retain:apply <name>` to write its requirements and tasks and build it, now or later from any checkout of this repository.

## Reporting

- Say which step you are on and what the commands returned. A part that failed validation failed; show its issues.
- Never say the design is in Retain unless `create` succeeded.
