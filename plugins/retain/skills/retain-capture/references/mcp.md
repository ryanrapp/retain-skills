# retain-capture: Retain MCP tools

Saving to Retain. Use the MCP tools when the Retain connector is available, otherwise the retain CLI.

These are the tools of the Retain MCP connector. Your host may show them with a prefix for the connector; the names below are the connector's own.

- Every tool returns structured content and the same JSON as text. A failure comes back as an error result that names the field and the correction; read it and fix the call instead of repeating it.
- Reads never change anything. Writes (`create_task`, `update_task`, `add_thread_update`, `import_notes`) are for what the user asked or confirmed. A repeated identical write is applied once.
- If the connector asks the user to sign in or approve, wait for them; never ask for a token.

| To | Tool | Arguments (`?` is optional) |
|---|---|---|
| Find an existing item first, so a save updates it instead of duplicating it | `search` | `query`, `limit?` |
| See the user's open tasks and thread suggestions before adding a task | `list_tasks` | `status?`, `cursor?`, `limit?` |
| See which threads need the user's attention (the weekly review) | `list_threads` | `section?`, `cursor?`, `limit?` |
| Read a thread before reporting on it | `fetch` | `id`, `detail?`, `at?`, `around?`, `window?` |
| Add a task the user asked for | `create_task` | `title`, `description?`, `priority?`, `due_at?` |
| Change a task, for example mark it done | `update_task` | `id`, `title?`, `description?`, `status?`, `priority?`, `due_at?`, `if_version?` |
| Record something the user confirmed on a thread (a decision, something done, an outcome) | `add_thread_update` | `thread_id`, `kind`, `text`, `actor?`, `artifact_url?`, `occurred_at?`, `origin?`, `host?`, `authorization` |
| Import the user's own notes or a transcript as source material | `import_notes` | `title`, `text`, `format?`, `occurred_at?` |

Not available as MCP tools: read a thread's recorded updates; upload a file for an import to attach. Do without, or use the CLI where it is installed.

## What each tool is for

- `search`: Find passages across the user's meetings, memories and threads. Use when prior conversations, people, projects or commitments matter. Returns ids for fetch and, for a recording, who said it and when (citation). A snippet is a pointer: fetch the source before relying on it. Not for general knowledge.
- `list_tasks`: What the user has to do: their open tasks plus the actions suggested on their threads, one list, most urgent first. Use for what's on their plate or outstanding. Retain maintains it: never offer to clean up, deduplicate, archive or bulk-edit. Change an item only when asked.
- `list_threads`: List the user's threads with status and likely next step, Action owed first. Use to see what is going on in a thread or to pick one to fetch. A thread with briefing_url has a short audio briefing the user can play. For what the user has to do, use list_tasks.
- `fetch`: Read one item by the id search or a list returned (thread:, memory:, task:, action:). A thread comes with its status and verbatim quotes. A meeting comes as its summary; read a stretch with at (seconds) or around (words), or detail full for all of it. Not for finding things; use search.
- `create_task`: Add a task for the user. Use only when they ask to add or track something. Look with list_tasks first so you do not duplicate one. An identical repeat creates nothing new.
- `update_task`: Change fields of an existing task, such as marking it done. Use only when the user asks. Pass if_version from list_tasks to avoid overwriting a newer edit.
- `add_thread_update`: Record something the user confirmed on a thread: a decision, an action taken, an outcome. Use after they say it happened or ask you to save it; a draft stays a draft. Not for notes or transcripts they wrote; use import_notes.
- `import_notes`: Import notes, a transcript or text the user wrote, said or pasted, as a source Retain interprets. Do not use for your own summaries, drafts or conclusions; use add_thread_update for those. Files stay in the CLI.
