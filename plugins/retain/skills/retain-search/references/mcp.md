# retain-search: Retain MCP tools

Reading Retain. Use the MCP tools when the Retain connector is available, otherwise the retain CLI.

These are the tools of the Retain MCP connector. Your host may show them with a prefix for the connector; the names below are the connector's own.

- Every tool returns structured content and the same JSON as text. A failure comes back as an error result that names the field and the correction; read it and fix the call instead of repeating it.
- Reads never change anything. Writes (`create_task`, `update_task`, `add_thread_update`, `import_notes`) are for what the user asked or confirmed. A repeated identical write is applied once.
- If the connector asks the user to sign in or approve, wait for them; never ask for a token.

| To | Tool | Arguments (`?` is optional) |
|---|---|---|
| Find passages across meetings, memories and threads | `search` | `query`, `limit?` |
| Read one item by the reference a search or list returned (thread:, memory:, task:, action:) | `fetch` | `id`, `detail?`, `at?`, `around?`, `window?` |
| See what is going on in the user's threads, Action owed first | `list_threads` | `section?`, `cursor?`, `limit?` |
| See what the user has to do: open tasks and the actions suggested on their threads | `list_tasks` | `status?`, `cursor?`, `limit?` |

Not available as MCP tools: read one thread's record; read one task; see what retain knows about the linked work in a folder before working on it. Do without, or use the CLI where it is installed.

## What each tool is for

- `search`: Find passages across the user's meetings, memories and threads. Use when prior conversations, people, projects or commitments matter. Returns ids for fetch and, for a recording, who said it and when (citation). A snippet is a pointer: fetch the source before relying on it. Not for general knowledge.
- `fetch`: Read one item by the id search or a list returned (thread:, memory:, task:, action:). A thread comes with its status and verbatim quotes. A meeting comes as its summary; read a stretch with at (seconds) or around (words), or detail full for all of it. Not for finding things; use search.
- `list_threads`: List the user's threads with status and likely next step, Action owed first. Use to see what is going on in a thread or to pick one to fetch. A thread with briefing_url has a short audio briefing the user can play. For what the user has to do, use list_tasks.
- `list_tasks`: What the user has to do: their open tasks plus the actions suggested on their threads, one list, most urgent first. Use for what's on their plate or outstanding. Retain maintains it: never offer to clean up, deduplicate, archive or bulk-edit. Change an item only when asked.
