---
name: retain-capture
description: Saves material to the user's Retain, namely their own notes or transcripts, tasks, and confirmed updates on a thread, and runs the weekly check-off of what they owe. Use when the user asks to import, save, log or track something, to add or update a task, or confirms that something on a thread is done. Do not use to save the assistant's own summaries, conclusions or drafts as notes, and do not use to look things up; the retain-search skill does that.
compatibility: Requires the Retain MCP connector or the retain CLI.
---

# Capture to Retain

Write to Retain through the Retain MCP tools when the connector is available, otherwise through the `retain` CLI. `references/mcp.md` and `references/cli.md` give the exact names for each.

## What belongs where

- **Import** turns what it is given into evidence that later searches return as something the user said or wrote. Use it only for material the user wrote, said or pasted. Never import your own summaries, drafts or conclusions.
- **A thread update** records what the user confirmed while working with you: a decision, something done, a result. It goes on the thread it is about. Use the kind `draft` for a draft, which is not something done, and report someone else's move as theirs, not the user's.
- **A task** is something the user asks to add or track.

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
