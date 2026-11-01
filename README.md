<p align="center">
  <a href="https://githubcard.com">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://githubcard.com/brand/githubcard-lockup-white.svg">
      <img src="https://githubcard.com/brand/githubcard-lockup.svg" alt="GitHubCard" height="48">
    </picture>
  </a>
</p>

# GitHubCard MCP server

Let your AI assistant create [GitHubCard](https://githubcard.com) images: profile
and repository cards for READMEs, link previews and social posts, with live
GitHub data.

- **Hosted:** `https://githubcard.com/mcp` (Streamable HTTP). Nothing to install.
- **Local:** `npx -y @ghcardhq/ghcard mcp` (stdio), from the
  [`ghcard` CLI](https://github.com/ghcardhq/ghcard). It can also edit a README on
  your disk.

Ask things like:

> Make a GitHubCard for my profile and add it to the top of my README.
>
> Create a dark card for `vercel/next.js` I can use as a link preview.
>
> Use the compact repo template, but with a `#1e3a5f` background and without the
> language widget.

## Get an API key

Creating cards needs a free API key. Sign in at
[githubcard.com/dashboard](https://githubcard.com/dashboard) and create one under
**API keys**. It looks like `ghc_live_…`.

Without a key, the server still lists templates, palettes and widget types, so
you can try it before signing up.

## Setup

Replace `ghc_live_…` with your key in the examples below.

### Claude Code

```bash
claude mcp add --transport http githubcard https://githubcard.com/mcp \
  --header "Authorization: Bearer ghc_live_…"
```

There is also a Claude Code plugin that adds the server together with a
GitHubCard skill: see [ghcardhq/skills](https://github.com/ghcardhq/skills).

### Claude Desktop

Settings → Connectors → **Add custom connector**, URL
`https://githubcard.com/mcp`. Custom headers are not available there yet, so to
pass your key, use the local server in `claude_desktop_config.json` instead:

```json
{
  "mcpServers": {
    "githubcard": {
      "command": "npx",
      "args": ["-y", "@ghcardhq/ghcard", "mcp"],
      "env": { "GHCARD_API_KEY": "ghc_live_…" }
    }
  }
}
```

### Cursor

`~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "githubcard": {
      "url": "https://githubcard.com/mcp",
      "headers": { "Authorization": "Bearer ghc_live_…" }
    }
  }
}
```

### VS Code

`.vscode/mcp.json` (VS Code asks for the key once and stores it securely):

```json
{
  "inputs": [
    {
      "id": "githubcard-key",
      "type": "promptString",
      "description": "GitHubCard API key",
      "password": true
    }
  ],
  "servers": {
    "githubcard": {
      "type": "http",
      "url": "https://githubcard.com/mcp",
      "headers": { "Authorization": "Bearer ${input:githubcard-key}" }
    }
  }
}
```

### Windsurf

`~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "githubcard": {
      "serverUrl": "https://githubcard.com/mcp",
      "headers": { "Authorization": "Bearer ghc_live_…" }
    }
  }
}
```

### Codex CLI

`~/.codex/config.toml`:

```toml
[mcp_servers.githubcard]
url = "https://githubcard.com/mcp"
bearer_token_env_var = "GHCARD_API_KEY"
```

Then export `GHCARD_API_KEY=ghc_live_…` in your shell.

### Any other client

Remote: Streamable HTTP at `https://githubcard.com/mcp`, header
`Authorization: Bearer ghc_live_…`.

Local (stdio): command `npx`, args `["-y", "@ghcardhq/ghcard", "mcp"]`, env
`GHCARD_API_KEY`. Add `--allow-file-writes` to the args to enable
`update_readme`. Needs Node.js 20 or later.

## Tools

| Tool | Key | What it does |
| --- | --- | --- |
| `create_card` | yes | Create (or reuse) a card for a user or `owner/repo`. Returns SVG/PNG links and Markdown/HTML embed snippets. Uses one unit of the daily quota |
| `validate_card` | yes | Dry run of `create_card`: checks ids, Pro features and custom designs, uses no quota |
| `list_templates` | no | Layouts, filterable by scope (profile/repo), Free plan and widget types |
| `list_palettes` | no | Background palettes |
| `list_components` | no | Widget types a card can contain |
| `get_template_design` | no | A template's layout as an editable design document |
| `get_card_design` | yes | The design document of one of your cards |
| `list_cards` | yes | Cards you have created |
| `delete_card` | yes | Delete a card; its links stop working |
| `get_usage` | yes | Your plan and today's remaining quota |
| `update_readme` | local only | Insert or update the card in a Markdown file. Only with `ghcard mcp --allow-file-writes` |

`create_card` is idempotent: the same inputs return the same card and link. Card
images always show current GitHub data, so there is no need to recreate them.

Failed calls return `isError: true` with a stable `code`, a `retryable` flag
and a `hint` telling the model what to do next.

## Limits

| Plan | Cards per day |
| --- | --- |
| Free | 100 |
| Pro | 5000 |

The quota resets at 00:00 UTC. Failed calls are not counted. Free cards carry a
small watermark. See [pricing](https://githubcard.com/~pricing).

## Links

- [githubcard.com](https://githubcard.com): the editor and gallery
- [ghcardhq/ghcard](https://github.com/ghcardhq/ghcard): the CLI and local MCP server
- [ghcardhq/skills](https://github.com/ghcardhq/skills): Claude Code plugin and skill
- [`server.json`](server.json): the entry in the
  [MCP registry](https://registry.modelcontextprotocol.io), as `com.githubcard/mcp`

## License

MIT
