---
name: mcp-server-builder
description: Build complete, production-ready MCP (Model Context Protocol) servers in Node.js/TypeScript or Python. Use this skill whenever building an MCP server, adding MCP tools to an existing service, debugging MCP connectivity, designing MCP tool schemas, implementing stdio or HTTP transport, or making any service accessible to AI coding agents like Claude Code or Cursor. Triggers on: "build an MCP server", "MCP tool", "make this work with Claude Code", "connect to Cursor", "MCP for my API", "add MCP support", "Model Context Protocol", "stdio transport", "MCP tool schema", "expose tools to agents". Always use this skill for any MCP work — it contains verified patterns for the most common failure modes.
---

# MCP Server Builder Skill

Build production-ready MCP servers that expose tools to coding agents. This skill covers the complete workflow from architecture to deployment, with verified patterns for every common failure mode.

## Critical Rules (Read First)

These are the mistakes that break MCP servers silently:

1. **NEVER write to stdout in stdio servers.** `console.log()` in any file imported by `mcp.js` corrupts the JSON-RPC stream. Use `console.error()` for ALL debug output.
2. **Tool descriptions are agent instructions.** Write them as if telling a human what the tool does and when to use it.
3. **Input schemas must be complete.** Every parameter needs a description. Agents use descriptions to decide what to pass.
4. **Errors must be returned, not thrown.** Return `{ isError: true }` responses — don't throw uncaught exceptions.
5. **stdio is for local. HTTP is for remote.** Don't serve stdio over a network.

---

## Architecture Decision: Transport

### stdio Transport (V1 default — local tools)

Use when: The MCP server runs on the user's machine alongside Claude Code / Cursor.

```
Claude Code → spawn process → mcp.js → your API
```

Setup in `claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "your-server": {
      "command": "node",
      "args": ["/absolute/path/to/src/mcp.js"],
      "env": {
        "YOUR_API_KEY": "key-here",
        "API_URL": "http://localhost:3000"
      }
    }
  }
}
```

### Streamable HTTP Transport (hosted — remote tools)

Use when: The MCP server is hosted on a server and accessed over the network.

```
Claude Code → HTTPS → your hosted MCP endpoint
```

Config:
```json
{
  "mcpServers": {
    "your-server": {
      "url": "https://api.yourservice.com/mcp",
      "headers": { "Authorization": "Bearer your-key" }
    }
  }
}
```

---

## Node.js / TypeScript Implementation

### Project Structure

```
mcp-server/
├── package.json
├── tsconfig.json         (if TypeScript)
├── src/
│   ├── mcp.ts            — MCP server entry point (stdio)
│   ├── tools/
│   │   ├── analyze.ts    — One file per logical tool group
│   │   ├── library.ts
│   │   └── compare.ts
│   ├── client.ts         — API client (auth, retry, error handling)
│   └── types.ts          — Shared TypeScript types
└── dist/                 — Compiled output (TypeScript only)
```

### package.json

```json
{
  "name": "your-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "bin": { "your-mcp": "dist/mcp.js" },
  "scripts": {
    "build": "tsc",
    "start": "node dist/mcp.js",
    "dev": "tsx src/mcp.ts"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.10.0",
    "zod": "^3.23.0"
  },
  "devDependencies": {
    "typescript": "^5.4.0",
    "tsx": "^4.0.0",
    "@types/node": "^20.0.0"
  }
}
```

### Core Server Pattern (TypeScript)

```typescript
// src/mcp.ts
#!/usr/bin/env node

// CRITICAL: Never import anything that uses console.log before this line
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
  ErrorCode,
  McpError,
} from '@modelcontextprotocol/sdk/types.js';
import { z } from 'zod';

const server = new Server(
  { name: 'your-server', version: '1.0.0' },
  { capabilities: { tools: {} } }
);

// ─── Tool Definitions ────────────────────────────────────────────────────────

const TOOLS = [
  {
    name: 'analyze_site',
    description:
      'Analyzes a live website and returns a structured UI Skill JSON. ' +
      'Use this when you need design patterns from a specific URL. ' +
      'Takes 15–45 seconds for a fresh analysis, under 1 second if cached.',
    inputSchema: {
      type: 'object' as const,
      properties: {
        url: {
          type: 'string',
          description: 'Full URL to analyze, including https:// (e.g. https://stripe.com)',
        },
        page_type: {
          type: 'string',
          enum: ['homepage', 'pricing', 'docs', 'landing', 'dashboard'],
          description: 'Type of page to analyze. Defaults to homepage.',
        },
      },
      required: ['url'],
    },
  },
  {
    name: 'get_skill',
    description:
      'Fetches a pre-analyzed UI Skill from the library by domain name. ' +
      'Instant response — no scraping required. ' +
      'Use this when you need design patterns for a well-known site (e.g. linear.app, stripe.com).',
    inputSchema: {
      type: 'object' as const,
      properties: {
        domain: {
          type: 'string',
          description: 'Domain name without protocol (e.g. linear.app, stripe.com, notion.so)',
        },
      },
      required: ['domain'],
    },
  },
  {
    name: 'search_skills',
    description:
      'Search the skill library using natural language or tags. ' +
      'Use this when looking for design patterns by style, industry, or component type. ' +
      'Returns top matching skills with summaries.',
    inputSchema: {
      type: 'object' as const,
      properties: {
        query: {
          type: 'string',
          description: 'Natural language query e.g. "dark minimal developer tool" or "fintech with card layout"',
        },
        tags: {
          type: 'array',
          items: { type: 'string' },
          description: 'Filter by specific tags e.g. ["dark-mode", "minimalist"]',
        },
        limit: {
          type: 'number',
          description: 'Maximum results to return (default 5, max 20)',
        },
      },
    },
  },
] as const;

// ─── Tool Handlers ───────────────────────────────────────────────────────────

async function handleAnalyzeSite(args: { url: string; page_type?: string }) {
  // Validate URL
  try { new URL(args.url); } catch {
    return { content: [{ type: 'text', text: 'Invalid URL. Include https:// e.g. https://stripe.com' }], isError: true };
  }

  try {
    const response = await fetch(`${process.env.API_URL}/analyze`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json', 'x-api-key': process.env.API_KEY! },
      body: JSON.stringify({ url: args.url, page_type: args.page_type ?? 'homepage' }),
    });

    if (!response.ok) {
      const err = await response.json().catch(() => ({ error: response.statusText }));
      return { content: [{ type: 'text', text: `Analysis failed: ${err.error || response.statusText}` }], isError: true };
    }

    const skill = await response.json();
    return { content: [{ type: 'text', text: JSON.stringify(skill, null, 2) }] };
  } catch (err) {
    return { content: [{ type: 'text', text: `Network error: ${(err as Error).message}` }], isError: true };
  }
}

// ─── Request Handlers ────────────────────────────────────────────────────────

server.setRequestHandler(ListToolsRequestSchema, async () => ({ tools: TOOLS }));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case 'analyze_site':
      return handleAnalyzeSite(args as { url: string; page_type?: string });
    case 'get_skill':
      return handleGetSkill(args as { domain: string });
    case 'search_skills':
      return handleSearchSkills(args as { query?: string; tags?: string[]; limit?: number });
    default:
      throw new McpError(ErrorCode.MethodNotFound, `Unknown tool: ${name}`);
  }
});

// ─── Start Server ────────────────────────────────────────────────────────────

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error('MCP server running'); // stderr ONLY
}

main().catch((err) => {
  console.error('Fatal error:', err);
  process.exit(1);
});
```

---

## Python / FastMCP Implementation

### Project Structure

```
mcp-server/
├── pyproject.toml
├── src/
│   └── server.py         — FastMCP server
└── tests/
    └── test_tools.py
```

### pyproject.toml

```toml
[project]
name = "your-mcp-server"
version = "1.0.0"
requires-python = ">=3.11"
dependencies = ["fastmcp>=2.0.0", "httpx>=0.27.0"]

[project.scripts]
your-mcp = "src.server:main"
```

### Core Pattern (Python / FastMCP)

```python
# src/server.py
from fastmcp import FastMCP
from pydantic import BaseModel, Field
from typing import Optional
import httpx
import os

mcp = FastMCP("your-server")

# ─── Input Models ─────────────────────────────────────────────────────────────

class AnalyzeSiteInput(BaseModel):
    url: str = Field(description="Full URL including https:// (e.g. https://stripe.com)")
    page_type: Optional[str] = Field(
        default="homepage",
        description="Page type: homepage, pricing, docs, landing, dashboard"
    )

class GetSkillInput(BaseModel):
    domain: str = Field(description="Domain name without protocol (e.g. linear.app, stripe.com)")

class SearchSkillsInput(BaseModel):
    query: Optional[str] = Field(default=None, description="Natural language search query")
    tags: Optional[list[str]] = Field(default=None, description="Filter by tags")
    limit: Optional[int] = Field(default=5, description="Maximum results (default 5, max 20)")

# ─── Tools ───────────────────────────────────────────────────────────────────

@mcp.tool
async def analyze_site(input: AnalyzeSiteInput) -> str:
    """
    Analyzes a live website and returns structured UI Skill JSON.
    Use when you need design patterns from a specific URL.
    Takes 15–45 seconds fresh, under 1 second if cached.
    """
    async with httpx.AsyncClient(timeout=60) as client:
        response = await client.post(
            f"{os.environ['API_URL']}/analyze",
            headers={"x-api-key": os.environ["API_KEY"]},
            json={"url": input.url, "page_type": input.page_type}
        )
        if not response.is_success:
            return f"Error: {response.status_code} — {response.text}"
        return response.text

@mcp.tool
async def get_skill(input: GetSkillInput) -> str:
    """
    Fetches a pre-analyzed UI Skill from the library by domain.
    Instant response — no scraping. Use for well-known sites.
    """
    async with httpx.AsyncClient() as client:
        response = await client.get(
            f"{os.environ['API_URL']}/skills/{input.domain}",
            headers={"x-api-key": os.environ["API_KEY"]}
        )
        if response.status_code == 404:
            return f"No skill found for {input.domain}. Try analyze_site() to generate one."
        return response.text

# ─── Entry Point ─────────────────────────────────────────────────────────────

def main():
    mcp.run(transport="stdio")

if __name__ == "__main__":
    main()
```

---

## Tool Design Principles

### Writing Good Tool Descriptions

The tool description is the agent's primary signal for when and how to use the tool. Write it as three sentences:

1. **What it does** — one sentence, active voice
2. **When to use it** — specific trigger contexts
3. **Important caveats** — timing, side effects, limits

```
❌ "Gets a skill."

✅ "Fetches a pre-analyzed UI Skill from the library by domain name.
   Use when you need design patterns for a well-known site (stripe.com, linear.app, notion.so).
   Returns instantly — no scraping required."
```

### Input Schema Design

Every parameter needs:
- `type` — string, number, boolean, array, object
- `description` — what it expects, format hints, examples
- `enum` — if there are valid values, list them

```typescript
// ❌ Under-specified
{ url: { type: 'string' } }

// ✅ Well-specified
{
  url: {
    type: 'string',
    description: 'Full URL including https:// protocol. Examples: https://stripe.com, https://linear.app/pricing'
  }
}
```

### Error Responses

Always return errors as tool responses, not thrown exceptions:

```typescript
// ❌ Throws — crashes the server or produces confusing output
throw new Error('API key invalid');

// ✅ Returns — agent can read and handle the error
return {
  content: [{ type: 'text', text: 'API key invalid. Check UISKILL_API_KEY in your MCP config.' }],
  isError: true
};
```

---

## Testing Your MCP Server

### MCP Inspector (interactive)

```bash
npx @modelcontextprotocol/inspector node src/mcp.js
```

Opens a browser UI where you can:
- List all tools
- Call tools interactively
- See raw JSON-RPC messages

### Manual stdio test

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' | node src/mcp.js
```

Expected: JSON response listing your tools (not silence, not an error).

### Common Failure Modes

| Symptom | Cause | Fix |
|---|---|---|
| Inspector shows no tools | Server exits immediately | Check for uncaught top-level errors in node src/mcp.js |
| Tools list but calls fail | Handler throws instead of returning error | Wrap handlers in try/catch, return `{ isError: true }` |
| Garbled JSON in inspector | console.log() used somewhere | Replace all console.log with console.error |
| "Spawn failed" in Claude Code | Wrong path in config | Use absolute paths, verify with node /absolute/path/to/mcp.js |
| Tool appears but does nothing | API_URL env var not set | Check env block in claude_desktop_config.json |

---

## Deployment (HTTP Transport)

For hosted MCP servers (accessible to any user, not just local):

```typescript
// src/mcp-http.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StreamableHTTPServerTransport } from '@modelcontextprotocol/sdk/server/streamableHttp.js';
import express from 'express';

const app = express();
app.use(express.json());

app.post('/mcp', async (req, res) => {
  const server = new Server({ name: 'your-server', version: '1.0.0' }, { capabilities: { tools: {} } });
  // ... register handlers ...
  const transport = new StreamableHTTPServerTransport({ sessionIdGenerator: undefined });
  await server.connect(transport);
  await transport.handleRequest(req, res, req.body);
});

app.listen(3000);
```

Authentication for hosted MCP:
```typescript
app.post('/mcp', (req, res, next) => {
  const key = req.headers.authorization?.replace('Bearer ', '');
  if (!key || !validKeys.has(key)) return res.status(401).json({ error: 'Unauthorized' });
  next();
}, mcpHandler);
```