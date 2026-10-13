# retain-email-search: Retain MCP tools

Reading the user's email. Use the MCP tools when the Retain connector is available, otherwise the retain CLI. Both work only where email search is enabled.

These are the tools of the Retain MCP connector. Your host may show them with a prefix for the connector; the names below are the connector's own.

- Every tool returns structured content and the same JSON as text. A failure comes back as an error result that names the field and the correction; read it and fix the call instead of repeating it.
- Reads never change anything. Writes (`create_task`, `update_task`, `add_thread_update`, `import_notes`) are for what the user asked or confirmed. A repeated identical write is applied once.
- If the connector asks the user to sign in or approve, wait for them; never ask for a token.

| To | Tool | Arguments (`?` is optional) |
|---|---|---|
| List the connected Gmail accounts and whether each can be searched | `list_email_accounts` |  |
| Search by Gmail syntax (native), by meaning (semantic) or both (hybrid), with hard constraints | `search_emails` | `mode?`, `gmail_query?`, `semantic_text?`, `original_question?`, `account_ids?`, `from?`, `to?`, `after?`, `before?`, `subject_terms?`, `include_spam_trash?`, `limit?`, `cursor?` |
| Read one email by the ref a search returned | `read_email` | `ref` |
| Read the messages around a hit in its Gmail conversation | `read_email_conversation` | `thread_ref`, `match_ref?`, `cursor?`, `limit?` |

## What each tool is for

- `list_email_accounts`: The user's connected Gmail accounts and whether each can be searched. Use when the account is ambiguous or a search reports one unavailable.
- `search_emails`: Search the user's email. native (default): Gmail syntax in gmail_query. semantic: a description in semantic_text, matched by meaning in the indexed part of the mailbox. hybrid: both. Returns refs for read_email, never an answer; one page is not the mailbox. Check accounts, semantic, partial.
- `read_email`: Read one email in full by ref from search_emails. A snippet is only a pointer: read before quoting or relying on it. Attachment contents are not read, only listed.
- `read_email_conversation`: Read the messages around a search hit in its Gmail conversation, oldest first, each with its own ref to cite. Use when a short reply needs its context or a later message may revise an earlier one.
