---
name: retain-capture
description: Saves to the user's Retain their own notes or transcripts, tasks and confirmed thread updates, links work in progress on their computer to the thread it advances, and runs the weekly check-off of what they owe. Use when the user asks to import, save, log or track something, to save or report progress on a file they are working on (a proposal, a plan, an OpenSpec change), to add or update a task, or confirms that something is done. Do not use to save the assistant's own summaries, conclusions or drafts as notes, and do not use to look things up; the retain-search skill does that.
compatibility: Requires the Retain MCP connector or the retain CLI.
---

# Capture to Retain

Write to Retain through the Retain MCP tools when the connector is available, otherwise through the `retain` CLI. `references/mcp.md` and `references/cli.md` give the exact names for each.

## What belongs where

- **A file the user is working on** (a proposal, plan or spec in their project) is linked, or imported as a draft copy, never imported as notes. Ask which before anything else, including before looking for the file; see below.
- **Import** turns what it is given into evidence that later searches return as something the user said or wrote. Use it only for material the user wrote, said or pasted. Never import your own summaries, drafts or conclusions.
- **A thread update** records what the user confirmed while working with you: a decision, something done, a result. It goes on the thread it is about. Use the kind `draft` for a draft, which is not something done, and report someone else's move as theirs, not the user's.
- **A task** is something the user asks to add or track.

## Work in progress on the user's computer

Work the user is doing in files (an OpenSpec change, a plan with checkboxes) can be linked to the thread it advances, so Retain sees how it moves without anyone retyping it.

- **If it is still changing on disk, link it** with `retain work link` in its folder. Without `--to` it lists what is there and the threads each could advance; pick with the user, then link with `--to <thread>`, adding `--action <id>` when it serves one of the thread's owed actions. Linking needs the user's request or yes, like any save.
- **If the user wants Retain to hold a copy** (to read it on their phone, say), import it as a draft. A work file is never imported as notes: it is not something the user said in a meeting.
- After linking, Retain records how the work moves once a day by itself. Progress on a work file (tasks checked off, a draft further along) is never a thread update, even when the user asks you to tell Retain about it: if the work is linked, Retain already has it; if it is not, offer to link it. Add a thread update only for a decision or outcome the user confirms.
- Do not go looking for the file before deciding how it belongs in Retain: offering to link it, or to import a copy as a draft, comes first.

## Saving only what was asked or confirmed

- Save when the user asks, or when they confirm what you proposed. An explicit request needs no second question; a proposal waits for a yes.
- Brainstorming, options you floated and ideas the user did not adopt are not saved.
- Record the authorization honestly: the user asked you to save it, or they approved your proposal. It is your statement of what you were told, not proof of consent, and you never present it as verified.
- Look first (search, or the list of open tasks) so a save updates an existing item instead of duplicating it. When the thread to update is not obvious, ask which one.
- Retrying a failed save uses the same request id, so it is applied once.
- Long text goes in a file or on stdin, not on a command line.

## The weekly check-off

When the user wants to review what they owe:

1. List the threads that need their attention (Action owed first) and what they have to do.
2. Go through the items with the user and ask which are done. Do not assume.
3. For each item the user confirms, add an outcome update on its thread, as the user's own move.
4. Read the thread back and tell the user where it now stands. A thread leaves Action owed only when Retain moves it; say what the tool returned.
5. Items the user did not confirm stay as they are. Report as done only what they confirmed.

## Reporting back

Report only what a tool returned. Say what was saved, with its title and reference. If a call failed, say it failed. Do not claim a save you did not make, and do not say a thread is resolved unless its returned section says so.
