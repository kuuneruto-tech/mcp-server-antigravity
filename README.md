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

## Troubleshooting / Manual Setup

If `npx` doesn't work, you can build the server manually in a few steps.

### Option A: Ask Claude to fix it

If you're using Claude Code, just say:

> "The antigravity MCP server isn't working. Fix it."

Claude can read this README, recreate the server from the source below, and patch your `~/.mcp.json` automatically.

### Option B: Build it yourself

1. Create a folder and save the source code below as `index.js`
2. Run `npm install @modelcontextprotocol/sdk zod`
3. Update `~/.mcp.json` to point to the folder:

```json
{
  "mcpServers": {
    "antigravity": {
      "command": "node",
      "args": ["/path/to/your/folder/index.js"]
    }
  }
}
```

### Full source (`index.js`)

```js
#!/usr/bin/env node
const { McpServer } = require("@modelcontextprotocol/sdk/server/mcp.js");
const { StdioServerTransport } = require("@modelcontextprotocol/sdk/server/stdio.js");
const { z } = require("zod");
const { spawn, execSync } = require("child_process");

function findAgy() {
  const custom = process.env.AGY_PATH;
  if (custom) return custom;
  try {
    return execSync("which agy", { encoding: "utf8" }).trim();
  } catch {
    throw new Error("Antigravity CLI (agy) not found. Install it or set AGY_PATH env var.");
  }
}

const AGY_BIN = findAgy();

const server = new McpServer({
  name: "antigravity",
  version: "1.0.0",
});

function runAgy(args, timeout = 120000) {
  return new Promise((resolve, reject) => {
    const proc = spawn(AGY_BIN, args, {
      stdio: ["ignore", "pipe", "pipe"],
    });
    let stdout = "";
    let stderr = "";
    proc.stdout.on("data", (d) => { stdout += d; });
    proc.stderr.on("data", (d) => { stderr += d; });
    const timer = setTimeout(() => {
      proc.kill();
      reject(new Error("agy timed out after " + timeout + "ms"));
    }, timeout);
    proc.on("close", (code) => {
      clearTimeout(timer);
      if (code !== 0 && !stdout) {
        reject(new Error(stderr || "agy exited with code " + code));
      } else {
        resolve(stdout || stderr || "(no response)");
      }
    });
  });
}

const DEPTH_PREFIX = {
  low:  "Answer briefly and directly. No need for deep reasoning.",
  high: "Think step by step very carefully before answering.",
};

server.tool(
  "ask_antigravity",
  `Ask Antigravity (Gemini-based AI) a question or coding task.

Use Antigravity FOR (Claude's weak points):
- Web search / real-time info: search_web tool (Google Search)
- Image generation: generate_image → saves PNG to brain folder, returns path
- Reading URLs / web pages: read_url_content
- Huge codebase analysis: 1M token context (vs Claude's 200k)
- Viewing images, PDFs, audio, video files: view_file
- Parallel sub-agents: define_subagent + invoke_subagent

Do NOT use Antigravity for:
- Precise code editing (use Claude's Edit/Write tools instead)
- Git operations
- Interactive debugging sessions
- Tasks already solvable with Claude's own tools

thinking_depth:
- "low"  : quick answer, minimal reasoning (simple Q&A, lookups)
- "high" : deep step-by-step reasoning (architecture, hard bugs, analysis)
- omit   : let Antigravity decide`,
  {
    prompt: z.string().describe("The question or task to send to Antigravity"),
    thinking_depth: z.enum(["low", "high"]).optional().describe("low = quick, high = deep reasoning"),
  },
  async ({ prompt, thinking_depth }) => {
    const prefix = thinking_depth ? DEPTH_PREFIX[thinking_depth] + "\n\n" : "";
    const output = await runAgy(["--print", prefix + prompt]);
    return { content: [{ type: "text", text: output }] };
  }
);

const transport = new StdioServerTransport();
server.connect(transport);
```

## License

MIT
