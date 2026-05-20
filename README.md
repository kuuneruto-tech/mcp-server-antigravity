# mcp-server-antigravity

MCP server that connects Claude Code to [Google Antigravity](https://antigravity.google) (`agy` CLI), filling in Claude's weak spots with Gemini's strengths.

## What this adds to Claude

| Capability | Tool used |
|---|---|
| Web search (Google) | `search_web` |
| Image generation | `generate_image` |
| Read URLs / web pages | `read_url_content` |
| View images, PDFs, audio, video | `view_file` |
| 1M token context (huge codebases) | Gemini's context window |
| Parallel sub-agents | `define_subagent` + `invoke_subagent` |
| YouTube transcript extraction | `run_command` + youtube-transcript-api |

## Requirements

- [Antigravity CLI (`agy`)](https://antigravity.google) installed and authenticated
- Node.js 18+

## Installation

```bash
npx mcp-server-antigravity
```

Or add to your Claude Code MCP config (`~/.mcp.json`):

```json
{
  "mcpServers": {
    "antigravity": {
      "command": "npx",
      "args": ["mcp-server-antigravity"]
    }
  }
}
```

If `agy` is not in your PATH, set the `AGY_PATH` environment variable:

```json
{
  "mcpServers": {
    "antigravity": {
      "command": "npx",
      "args": ["mcp-server-antigravity"],
      "env": {
        "AGY_PATH": "/path/to/agy"
      }
    }
  }
}
```

## Tool: `ask_antigravity`

### Parameters

| Parameter | Type | Description |
|---|---|---|
| `prompt` | string | The question or task to send to Antigravity |
| `thinking_depth` | `"low"` \| `"high"` | `low` = quick answer, `high` = deep step-by-step reasoning |

### thinking_depth guide

- **`low`** — simple Q&A, lookups, quick searches
- **`high`** — architecture decisions, complex debugging, deep analysis
- **omit** — let Antigravity decide

## License

MIT
