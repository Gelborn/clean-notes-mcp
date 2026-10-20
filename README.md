<!-- mcp-name: app.cleannotes/clean-notes -->

<img src="https://cleannotes.app/icon.svg" width="56" height="56" alt="Clean Notes">

# Clean Notes MCP server

Kanban tasks and project notes for AI clients, with OAuth and per-project read or
write grants.

[Clean Notes](https://cleannotes.app/en) is a project workspace for small teams: a
kanban board with list and timeline views, project notes beside the tasks they
explain, and realtime updates. Its MCP server lets GitHub Copilot, ChatGPT, Claude
or any MCP client read and update the projects you approve, and nothing else.

[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Server-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=clean-notes&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fcleannotes.app%2Fmcp%22%7D)
[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install_Server-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=clean-notes&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fcleannotes.app%2Fmcp%22%7D&quality=insiders)

This repository documents the hosted server. There is nothing to install or run
locally: you point your client at the endpoint and sign in.

| | |
| --- | --- |
| Endpoint | `https://cleannotes.app/mcp` |
| Transport | Streamable HTTP, stateless |
| Authentication | OAuth 2.1 with dynamic client registration and PKCE. No API key or token to paste. |
| Tools | 11: six read, five write, no delete |
| Registry name | `app.cleannotes/clean-notes` |

You need a Clean Notes account. The free plan includes AI access.

## Configure

### VS Code with GitHub Copilot

Use the install button above, or add the server to `.vscode/mcp.json` in your
workspace (or to your user `mcp.json`):

```json
{
  "servers": {
    "clean-notes": {
      "type": "http",
      "url": "https://cleannotes.app/mcp"
    }
  }
}
```

Start the server. When VS Code asks to sign in to Clean Notes, allow it; your
browser opens Clean Notes, where you sign in and choose the projects to share.
The tools then appear in Copilot Chat in agent mode.

### Claude Code

```bash
claude mcp add --transport http clean-notes --scope user https://cleannotes.app/mcp
```

Then run `/mcp` in Claude Code and choose **Authenticate** next to Clean Notes.

### Claude (app and web)

**Settings → Connectors → Add custom connector**, paste
`https://cleannotes.app/mcp`, and authenticate.

### ChatGPT

Open the [Clean Notes plugin](https://chatgpt.com/plugins/plugin_asdk_app_6a7e93c1aac481918064f9df45173628),
connect your account and authorize your projects.

### Any other MCP client

Register `https://cleannotes.app/mcp` as a Streamable HTTP server and finish the
OAuth sign-in in your browser. The client discovers everything else from
`https://cleannotes.app/.well-known/oauth-protected-resource`.

## Choose what the assistant can reach

Signing in proves who you are; it does not open your workspace. Right after the
sign-in, Clean Notes asks which projects this connection may reach and, for each
one, whether it may only read or also make changes. A connection with no project
grant reads nothing.

- Grants are made and revoked in Clean Notes, in each project's **AI tools**
  panel.
- Project owners decide whether members may connect their own AI tools.
- The assistant cannot delete anything, and cannot change members, settings or
  billing.

## Use it

Ask in plain language, for example:

- "Which tasks in my Launch project are overdue or blocked, and who owns each one?"
- "Create tasks in my Launch project from these meeting notes, with an owner and a due date for each."
- "Mark the pricing page task as done and write a project note with today's decisions."
- "What changed in the Website project since Monday?"

## Tools

| Tool | Access | What it does |
| --- | --- | --- |
| `clean_notes_list_projects` | read | Lists the projects this connection may reach. |
| `clean_notes_get_project` | read | Returns a project with its ordered statuses and the members who can be assigned. |
| `clean_notes_list_tasks` | read | Lists tasks, filtered by status, assignee or last change, with a cursor and a limit. |
| `clean_notes_get_task` | read | Returns a task with its full details. |
| `clean_notes_list_notes` | read | Lists notes as short previews, with a cursor. |
| `clean_notes_get_note` | read | Returns a note's whole body. |
| `clean_notes_create_task` | write | Creates a task: title, details, status, priority, assignee and dates. |
| `clean_notes_update_task` | write | Changes a task's fields. |
| `clean_notes_complete_task` | write | Marks a task as done. |
| `clean_notes_create_note` | write | Creates a note with a title and a body, optionally pinned. |
| `clean_notes_update_note` | write | Edits a note's title, body or pin. |

Write tools work only in projects granted read and change; anywhere else the call
is refused.

## Built to be retried safely

- Every write takes an idempotency key, so a repeated call returns the first
  result instead of creating a second task or note.
- Updates and completion carry the record's `expectedUpdatedAt`. If someone else
  changed it first, the call fails with a conflict instead of overwriting their
  edit, and the assistant re-reads before trying again.
- Every tool is closed-world: it reads and writes the authorized projects only and
  never reaches the internet.
- Each project keeps an activity log of AI changes, with the client, tool, time and
  outcome. It stores metadata, not the contents of the calls.

## Pricing

AI access is on every plan. Only successful changes count toward the monthly
allowance; reads, conflicts and rejected calls are free.

| Plan | Price | AI changes per month |
| --- | --- | --- |
| Free | $0 | 50 |
| Pro | $4.90 per month | 5,000 |
| Scale | $14.90 per month | 25,000 |

Billing follows the projects you create, so people you invite never pay for a
seat. See [pricing](https://cleannotes.app/en#precos).

## Links

- Documentation: [cleannotes.app/en/ai/mcp](https://cleannotes.app/en/ai/mcp)
- Website: [cleannotes.app](https://cleannotes.app/en)
- Support: [cleannotes.app/en/support](https://cleannotes.app/en/support) or hello@cleannotes.app
- [Privacy policy](https://cleannotes.app/en/privacy) · [Terms](https://cleannotes.app/en/terms)

Clean Notes is a proprietary hosted service. This repository holds its public
documentation only; the server's source code is not published here.
