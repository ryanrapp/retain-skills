# retain-sync: Retain MCP tools

Syncing with Retain. The sync itself needs the retain CLI; with only the Retain connector, read the digest.

These are the tools of the Retain MCP connector. Your host may show them with a prefix for the connector; the names below are the connector's own.

- Every tool returns structured content and the same JSON as text. A failure comes back as an error result that names the field and the correction; read it and fix the call instead of repeating it.
- Reads never change anything. Writes (`create_task`, `update_task`, `add_thread_update`, `import_notes`) are for what the user asked or confirmed. A repeated identical write is applied once.
- If the connector asks the user to sign in or approve, wait for them; never ask for a token.

| To | Tool | Arguments (`?` is optional) |
|---|---|---|
| Read Retain's digest without syncing (a past day with a date) | `get_digest` | `date?` |
| Record finished work the user confirmed as an outcome on its thread | `add_thread_update` | `thread_id`, `kind`, `text`, `actor?`, `artifact_url?`, `occurred_at?`, `origin?`, `host?`, `authorization` |
| Read a thread back after an update | `fetch` | `id`, `detail?`, `at?`, `around?`, `window?` |

Not available as MCP tools: send today's progress for linked work and get the digest back; link a suggested item to its thread, or dismiss the suggestion. Do without, or use the CLI where it is installed.

## What each tool is for

- `get_digest`: Retain's digest of the user's day: what they owe, what waits on others, how their linked work moved, and what to confirm. Use for a morning summary or 'what's my day look like'. Say when it was made; offer briefing_url as given. Not a list to edit; use list_tasks and fetch for details.
- `add_thread_update`: Record something the user confirmed on a thread: a decision, an action taken, an outcome. Use after they say it happened or ask you to save it; a draft stays a draft. Not for notes or transcripts they wrote; use import_notes.
- `fetch`: Read one item by the id search or a list returned (thread:, memory:, task:, action:). A thread comes with its status and verbatim quotes. A meeting comes as its summary; read a stretch with at (seconds) or around (words), or detail full for all of it. Not for finding things; use search.
