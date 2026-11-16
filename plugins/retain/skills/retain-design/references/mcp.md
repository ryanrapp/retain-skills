# retain-design: Retain MCP tools

Writing a Retain design. The design is written with the retain CLI: the parts are written in the packet's folder under the packet root, validated, and the packet is created in Retain once both pass. Retain is read with the MCP tools when the connector is available, otherwise with the CLI.

These are the tools of the Retain MCP connector. Your host may show them with a prefix for the connector; the names below are the connector's own.

- Every tool returns structured content and the same JSON as text. A failure comes back as an error result that names the field and the correction; read it and fix the call instead of repeating it.
- Reads never change anything. Writes (`create_task`, `update_task`, `add_thread_update`, `import_notes`) are for what the user asked or confirmed. A repeated identical write is applied once.
- If the connector asks the user to sign in or approve, wait for them; never ask for a token.

| To | Tool | Arguments (`?` is optional) |
|---|---|---|
| Find passages across meetings, memories and threads, earlier technical designs by the words or meaning of their body (scope designs), or threads, tasks or other documents Retain wrote with the user (scope documents) | `search` | `query`, `scope?`, `kind?`, `state?`, `since?`, `cursor?`, `limit?` |
| Read one item by the reference a search or list returned (thread:, memory:, task:, action:, asset:) | `fetch` | `id`, `offset?`, `detail?`, `at?`, `around?`, `window?` |

Not available as MCP tools: read where packets go for this checkout, and its repository folder; read the instruction and template for the proposal or the design; validate the packet folder; every issue has a rule, a message and a fix; check whether this repository already has a packet of the name; create the packet in retain from the folder, once the proposal and the design are valid. Do without, or use the CLI where it is installed.

## What each tool is for

- `search`: Find things in Retain. Default scope memories: meetings, notes, email; what happened, with who said it and when. Scope documents: what Retain wrote with the user; work product, never evidence. Scope designs: earlier technical designs, by words in their body. Also threads, tasks, all. Fetch a hit before relying on it.
- `fetch`: Read one item by an id search or a list returned (thread:, memory:, task:, action:, asset:). A meeting comes as its summary; read a stretch with at (seconds) or around (words). asset: is a document Retain wrote with the user: work product, naming the records to fetch for facts. Pass next_offset as offset to read on.
