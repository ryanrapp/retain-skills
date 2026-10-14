---
name: retain-sync
description: Syncs the work the user linked to their Retain threads and reads back Retain's daily digest as a short summary of their day. Use when the user asks to sync with Retain, for their morning summary or digest, or when a scheduled task runs it with "morning". Do not use to link new work or to save notes, decisions or tasks; the retain-capture skill does that.
compatibility: Requires the retain CLI for local work; with only the Retain MCP connector it reads the digest and nothing local.
---

# Sync with Retain

Retain keeps the user's threads (what they owe, who is waiting) and makes a digest of their day. This skill sends Retain how the work the user linked to those threads has moved, then reads the digest back. `references/cli.md` and `references/mcp.md` give the exact names.

## Two modes

- **Morning (unattended).** The request says `morning`, usually from a scheduled task. Run the sync, show the digest, and stop. Confirm nothing, link nothing, record nothing, and ask no questions: nobody is there to answer. Anything that needs the user stays listed for them.
- **Interactive.** The user asked. Run the sync, show the digest, then offer the items that need an answer, one at a time. Act only on an explicit yes.

## Steps

1. Run `retain sync`. It re-reads every project the user linked work in, sends today's progress, and returns `projects` and Retain's `digest`. On a host with only the Retain connector (no shell), call `get_digest` instead and say that local work is shown as last reported, not freshly read. On a computer where the CLI is missing or too old to have `sync`, say that first, with how to get it (`references/cli.md`), then show `get_digest` the same way: a missing CLI is a setup problem to tell the user about, not a reason to skip the sync quietly.
2. If a project is `missing`, say so in one line: its folder moved or was deleted.
3. Show the digest (below). If `digest` is null, say why from `reason`: digests are off, Retain does not know the user's time zone yet, or there is none for that day.
4. Interactive only, for each item in `confirm`:
   - **A suggested link** ("retain-work-sync could advance Agent work sync"): on yes, run `retain work link "<project path>/<where>" --to <thread> --origin agent --host <your host> --authorization user_confirmed`, using `where` from the sync's `projects`. On no, run the same with `--dismiss <thread>` so it is never suggested again.
   - **Finished work** ("retain-cli-release finished; mark Ship the Retain CLI done?"): on yes, add an `outcome` thread update saying what finished, as the user's own move, with `--authorization user_confirmed`. Then read the thread back and say where it now stands; say it is done only if the returned section says so.

## Showing the digest

At most about 15 lines, in this order, leaving out empty parts:

- The date and when Retain made it ("Tuesday's digest, made 7:00, updated 8:31").
- What the user owes, most pressing first, marking overdue or due today.
- What is waiting on others, with how long.
- Linked work that moved ("retain-cli-release: 11 of 30 tasks, +7"), and anything finished.
- Stalled work on threads the user owes.
- What needs an answer (the confirm items), and how many active items are not linked to anything.
- If the digest has a briefing link, offer it as given ("you can listen to it"); never rewrite or open it.

Link each thread by the reference or link the digest gives. The digest is Retain's; your rendering is your own words about it, so never import it or save it as a note or a thread update.

## Honesty

- Report what the tools returned. A sync that failed for a project failed; say so.
- A snapshot is a reading of the user's files, not something anyone said or decided. Never present progress as a decision or as done.
- Retain moves a thread out of what is owed; you do not. Say what the tool returned.
