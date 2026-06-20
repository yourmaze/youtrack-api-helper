---
name: youtrack-api-helper
description: Work with YouTrack through a local REST API helper script. Use when the user asks to inspect YouTrack issues, create issues, post comments, or update issue fields with token-based API access.
---

# YouTrack API Helper

## Core Rules

- Load credentials from `$YOUTRACK_ENV_FILE` or `~/.config/youtrack-api-helper/youtrack.env`.
- Never print, log, summarize, or paste the token. If it appears in command output, redact it.
- Prefer the YouTrack REST API over browser scraping.
- Reading issues, comments, fields, projects, and users is allowed without extra confirmation.
- Before creating an issue, posting a comment, changing fields/status/assignee/tags/links, or performing any other mutation, show the exact intended action and wait for explicit user confirmation such as "send", "create", "post", or a localized equivalent.
- If the user explicitly asks for an immediate mutation and the requested text/action is unambiguous, confirmation can be treated as already given.
- If API returns `401` or `403`, ask the user to refresh permissions or replace the token. Do not retry with passwords.

## Quick Start

Use the bundled helper for API calls:

```bash
./yt-api GET '/api/users/me?fields=id,login,fullName'
```

The helper sources `YOUTRACK_URL` and `YOUTRACK_TOKEN` from the secret env file, adds the required headers, and calls `curl`.

## Issue Reading Workflow

1. Extract the issue id from the URL or user text. Normalize only casing; keep project key and number intact.
2. Fetch a compact issue first:

```bash
./yt-api GET '/api/issues/PROJECT-123?fields=id,idReadable,summary,description,project(shortName,name),customFields(name,value(name,localizedName,presentation,text,id)),comments(id,text,author(login,fullName),created,updated)'
```

3. If the user asks about a focused comment from a URL fragment, fetch comments and identify the relevant one by id when possible.
4. Present useful task context in the user's language unless they ask otherwise: summary, description, key comments, current status/assignee if fetched, and concrete implementation implications.

## Mutations

For comments:

```bash
./yt-api POST '/api/issues/PROJECT-123/comments?fields=id,text' '{"text":"Comment text"}'
```

For issue creation, field changes, links, or commands, use the official `/api` endpoint that matches the operation. Fetch project and field metadata first if exact values are unclear.

Before mutation, show a draft like:

```text
I am going to post this comment to PROJECT-123:
...

Send it?
```

After successful mutation, report the issue id and what changed. Do not include bearer tokens or raw sensitive headers.

## Error Handling

- `404`: verify issue key casing and whether the user has access.
- `400`: inspect the response body; usually a field name/value or JSON shape is wrong.
- `401/403`: token missing, expired, revoked, or lacks permission.
- `502/5xx`: retry once after a short pause; if it repeats, report YouTrack/server availability issue.

## Useful Fields

Use `fields=` aggressively to keep responses small. Common field sets:

```text
id,idReadable,summary,description
comments(id,text,author(login,fullName),created,updated)
customFields(name,value(name,localizedName,presentation,text,id))
project(shortName,name)
```
