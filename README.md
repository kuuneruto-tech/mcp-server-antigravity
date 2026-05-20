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

Add to your Claude Code MCP config (`~/.mcp.json`):

```json
{
  "mcpServers": {
    "antigravity": {
      "command": "npx",
      "args": ["github:kuuneruto-tech/mcp-server-antigravity"]
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
      "args": ["github:kuuneruto-tech/mcp-server-antigravity"],
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

---

## Tip: Teach Claude when to use this tool

Adding a few lines to your `CLAUDE.md` helps Claude decide when to delegate to Antigravity vs handle things itself (web search, image generation, large context, etc.). The right wording depends on your workflow — adjust to fit your environment.

---

## Troubleshooting / Manual Setup

If `npx` doesn't work, just ask Claude Code:

> "The antigravity MCP server isn't working. Fix it."

Claude will read this README and rebuild the server automatically. For reference, the server is a Node.js MCP server (`@modelcontextprotocol/sdk` + `zod`) that wraps `agy --print <prompt>` with `stdio: ["ignore", "pipe", "pipe"]` — the `ignore` on stdin is required, otherwise `agy` hangs waiting for input. It exposes one tool `ask_antigravity` with `prompt` (string) and optional `thinking_depth` (`"low"` / `"high"`) that prepends a brevity or deep-reasoning instruction to the prompt. The `agy` binary is resolved via `AGY_PATH` env var or `which agy` fallback.

## Author

[くうねると](https://note.com/kuuneruto)

## License

MIT
