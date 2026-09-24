<p align="center">
  <a href="https://urlpipe.dev/mcp-server"><img src="icon.svg" alt="URLpipe" width="72" height="72"></a>
</p>

<h1 align="center">URLpipe MCP server</h1>

<p align="center">
  Give your agent a browser that runs the page's JavaScript before it reads it.
  <br>
  <a href="https://urlpipe.dev/mcp-server">Website</a> ·
  <a href="https://urlpipe.dev/docs/mcp">Docs</a> ·
  <a href="https://urlpipe.dev/pricing">Pricing</a>
</p>

---

A plain `fetch` tool does an HTTP GET. On a site that builds itself in the browser —
React, Vue, most docs sites and dashboards — that returns an empty shell, and the agent
either says it can't read the page or guesses what was on it.

URLpipe loads every page in real Chrome first, then hands your agent what it asked
for: the page as Markdown, a screenshot it can look at, its metadata, its console
errors or a Lighthouse audit.

- **Hosted.** Streamable HTTP at `https://urlpipe.dev/mcp`. Nothing to install or run.
- **One bearer token.** Can be read-only, limited to one project, or set to expire.
- **14 tools.** The nine API operations, plus results, request history, projects and usage.
- **Same credits as the HTTP API.** 1,000 free every month, no card. Cached results cost nothing.
- **Processed in the EU.** Fetching, rendering and storage happen in Europe.

This repository holds the server's registry entry ([`server.json`](server.json)) and
the setup for each client. The server itself runs at urlpipe.dev.

## Get a token

1. [Sign up](https://urlpipe.dev) and confirm your email address.
2. In the dashboard, open **Organization → MCP access** and create a token. Choose
   **Fetch pages and read results** for an agent that works normally, or **Read past
   results only** for one that must never spend credits.
3. Copy it — it is shown once.

Every client sends it as `Authorization: Bearer YOUR_TOKEN`.

## Connect your client

### Claude Code

```bash
claude mcp add --transport http urlpipe https://urlpipe.dev/mcp \
  --scope user \
  --header "Authorization: Bearer YOUR_TOKEN"
```

Or share it with your team in `.mcp.json`, each person setting their own `URLPIPE_TOKEN`:

```json
{
  "mcpServers": {
    "urlpipe": {
      "type": "http",
      "url": "https://urlpipe.dev/mcp",
      "headers": {
        "Authorization": "Bearer ${URLPIPE_TOKEN}"
      }
    }
  }
}
```

### Claude Desktop

Where your organization has request headers in **Customize → Connectors → Add custom
connector**, add `https://urlpipe.dev/mcp` with *No sign-in* and an `authorization`
header of `Bearer YOUR_TOKEN`. Otherwise use the open-source
[mcp-remote](https://github.com/geelen/mcp-remote) bridge in
`claude_desktop_config.json` (needs Node.js):

```json
{
  "mcpServers": {
    "urlpipe": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://urlpipe.dev/mcp",
        "--header",
        "Authorization:${URLPIPE_AUTH}",
        "--transport",
        "http-only"
      ],
      "env": {
        "URLPIPE_AUTH": "Bearer YOUR_TOKEN"
      }
    }
  }
}
```

### Cursor

`~/.cursor/mcp.json`, or `.cursor/mcp.json` in a project:

```json
{
  "mcpServers": {
    "urlpipe": {
      "url": "https://urlpipe.dev/mcp",
      "headers": {
        "Authorization": "Bearer ${env:URLPIPE_TOKEN}"
      }
    }
  }
}
```

### VS Code

`.vscode/mcp.json` — VS Code asks for the token once and stores it securely:

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "urlpipe-token",
      "description": "URLpipe MCP token",
      "password": true
    }
  ],
  "servers": {
    "urlpipe": {
      "type": "http",
      "url": "https://urlpipe.dev/mcp",
      "headers": {
        "Authorization": "Bearer ${input:urlpipe-token}"
      }
    }
  }
}
```

### Windsurf

`mcp_config.json`:

```json
{
  "mcpServers": {
    "urlpipe": {
      "serverUrl": "https://urlpipe.dev/mcp",
      "headers": {
        "Authorization": "Bearer ${env:URLPIPE_TOKEN}"
      }
    }
  }
}
```

### Cline

`cline_mcp_settings.json`:

```json
{
  "mcpServers": {
    "urlpipe": {
      "type": "streamableHttp",
      "url": "https://urlpipe.dev/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_TOKEN"
      },
      "disabled": false,
      "autoApprove": ["list_projects", "get_usage", "get_result"]
    }
  }
}
```

### Zed

`settings.json`:

```json
{
  "context_servers": {
    "urlpipe": {
      "url": "https://urlpipe.dev/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_TOKEN"
      }
    }
  }
}
```

Step-by-step pages for each client, with a first prompt to try and troubleshooting,
are at [urlpipe.dev/integrations](https://urlpipe.dev/integrations).

## Tools

| Tool | What it returns | Credits |
|---|---|---:|
| `list_projects` | The projects the token can reach. Start here: every fetching tool takes a `project_id`. | 0 |
| `get_usage` | What the plan allows, what is left this period, and what each operation costs. | 0 |
| `fetch_markdown` | The page's main content as clean Markdown, after its JavaScript ran. **The one to use for reading.** | 1 |
| `fetch_html` | The rendered DOM — what a browser sees, not the shell curl gets. | 1 |
| `capture_screenshot` | A full-page screenshot, returned as an image the model can look at. Viewport, device scale, dark mode, one element by selector, PNG/JPEG/WebP. | 1 |
| `console_logs` | `console.error` and `console.warn` during load, uncaught exceptions and unhandled promise rejections. | 1 |
| `lighthouse_audit` | A real Lighthouse audit: performance, accessibility, best practices, SEO and Core Web Vitals. | 2 |
| `extract_metadata` | Open Graph, Twitter card and page metadata reconciled into one object. | 5 |
| `extract_keywords` | The 5–15 terms that best represent the page, ranked by a language model. | 15 |
| `summarize_page` | A short Markdown summary of the page's main content. | 17 |
| `scrape_url` | Several of the above from one page visit. Each operation is billed as usual. | sum |
| `get_result` | The result of an earlier request, by its token. | 0 |
| `list_requests` | A project's recent requests, newest first. | 0 |
| `get_request` | Everything about one past request except its result: timings, cache, webhook delivery. | 0 |

**Async is the default,** as it is over HTTP: a fetching tool returns a token straight
away and `get_result` collects the result. Pass `sync: true` to wait for it instead.
Most agents work this out from the tool descriptions; saying "use sync: true" in the
prompt saves a round trip.

**A repeated request is free.** Results are kept for 30 days and reused while they are
fresher than `max_age` (7 days by default).

`initialize` and `tools/list` answer without a token, so you can see the list above
straight from the server:

```bash
curl -s https://urlpipe.dev/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## A first prompt

```text
Using urlpipe with sync: true, read https://news.ycombinator.com as Markdown and list
the five top stories with their links. Then take a screenshot of the first story's page
and tell me what the page looks like.
```

## What it doesn't do

URLpipe answers questions about one URL at a time. It doesn't crawl whole sites,
discover URLs, or reach `localhost` and private addresses — it fetches public pages
from its own servers. It follows each site's robots.txt by default.

## Links

- [MCP server overview](https://urlpipe.dev/mcp-server)
- [MCP docs](https://urlpipe.dev/docs/mcp) — tokens, scopes, limits
- [HTTP API docs](https://urlpipe.dev/docs) and [OpenAPI spec](https://urlpipe.dev/openapi.json)
- [Free tools](https://urlpipe.dev/tools) — try every operation without an account
- Questions: contact@urlpipe.dev

Licensed under MIT: the configuration and documentation in this repository.
The URLpipe service is covered by its [terms](https://urlpipe.dev/terms).
