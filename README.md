# MCP Playbook

> **Model Context Protocol** — the open standard that wires AI models to the real world.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![MCP Spec](https://img.shields.io/badge/MCP-Spec%201.0-blueviolet)](https://modelcontextprotocol.io)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Stars](https://img.shields.io/github/stars/avinashmyerolkar/mcp-playbook?style=social)](https://github.com/avinashmyerolkar/mcp-playbook)

---

## What is MCP?

**Model Context Protocol (MCP)** is an open protocol introduced by Anthropic in November 2024. It standardizes how AI applications connect to external data sources, tools, and services — replacing brittle, one-off integrations with a single, universal interface.

Think of MCP as **USB-C for AI**: one standard plug that connects any model to any tool.

```
Before MCP                         After MCP
──────────────────────────────     ──────────────────────────────
Claude  ──── custom glue ──► DB    Claude ──► MCP Client
GPT-4   ──── custom glue ──► FS           │
Gemini  ──── custom glue ──► API          ▼
                                   MCP Server ──► DB / FS / API
                                   (one protocol, any model)
```

---

## Why MCP Matters

| Without MCP | With MCP |
|---|---|
| Every integration is hand-rolled | One protocol for all integrations |
| Model-specific plugins | Model-agnostic servers |
| Context siloed inside the model | Context pulled from live systems |
| Hard to audit or compose | Composable, inspectable, secure |

---

## Repository Structure

```
mcp-playbook/
├── 01-concepts/            # Core MCP theory & architecture
│   ├── architecture.md
│   ├── transport-layers.md
│   └── primitives.md
├── 02-servers/             # Building MCP servers (Python & TypeScript)
│   ├── hello-world/
│   ├── filesystem-server/
│   └── database-server/
├── 03-clients/             # Connecting MCP clients
│   ├── claude-desktop/
│   └── custom-client/
├── 04-tools/               # Tool definitions & examples
├── 05-resources/           # Resource exposure patterns
├── 06-prompts/             # Prompt templates via MCP
├── 07-real-world/          # Production patterns & case studies
│   ├── auth-patterns.md
│   ├── error-handling.md
│   └── scaling.md
├── 08-integrations/        # Popular integrations (GitHub, Slack, DBs)
└── 09-recipes/             # Copy-paste recipes for common use cases
```

---

## Core Concepts

### The Three Primitives

MCP servers expose three types of capabilities:

```
┌─────────────────────────────────────────────────────────┐
│                     MCP Server                          │
│                                                         │
│  ┌──────────┐   ┌──────────────┐   ┌────────────────┐  │
│  │  Tools   │   │  Resources   │   │    Prompts     │  │
│  │          │   │              │   │                │  │
│  │ Functions│   │ Live data    │   │ Reusable       │  │
│  │ the model│   │ (files, DBs, │   │ prompt         │  │
│  │ can call │   │  APIs)       │   │ templates      │  │
│  └──────────┘   └──────────────┘   └────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

| Primitive | Model can... | Example |
|---|---|---|
| **Tools** | Execute actions | `run_query`, `send_email`, `create_issue` |
| **Resources** | Read live data | `file://`, `db://`, `api://` |
| **Prompts** | Use templates | `summarize_issue`, `review_pr` |

### Architecture

```
┌──────────────┐     JSON-RPC 2.0      ┌──────────────────┐
│  MCP Client  │◄────over stdio/SSE───►│   MCP Server     │
│              │                       │                  │
│ • Claude     │   initialize          │ • tools/list     │
│   Desktop    │──────────────────────►│ • tools/call     │
│ • Claude     │   tools/list          │ • resources/list │
│   Code       │──────────────────────►│ • resources/read │
│ • Custom     │   tools/call          │ • prompts/list   │
│   Apps       │──────────────────────►│ • prompts/get    │
└──────────────┘                       └──────────────────┘
```

### Transport Layers

| Transport | Use Case |
|---|---|
| **stdio** | Local processes (most common) |
| **SSE (HTTP)** | Remote servers, web deployments |
| **WebSocket** | Bidirectional, real-time use cases |

---

## Quick Start

### 1. Run a local MCP server (Python)

```bash
pip install mcp
```

```python
# server.py
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

app = Server("my-first-server")

@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="greet",
            description="Greet someone by name",
            inputSchema={
                "type": "object",
                "properties": {"name": {"type": "string"}},
                "required": ["name"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "greet":
        return [TextContent(type="text", text=f"Hello, {arguments['name']}!")]

async def main():
    async with stdio_server() as (r, w):
        await app.run(r, w, app.create_initialization_options())

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

### 2. Wire it into Claude Desktop

```json
// ~/Library/Application Support/Claude/claude_desktop_config.json
{
  "mcpServers": {
    "my-first-server": {
      "command": "python",
      "args": ["/path/to/server.py"]
    }
  }
}
```

### 3. TypeScript version

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server(
  { name: "my-first-server", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler("tools/list", async () => ({
  tools: [{
    name: "greet",
    description: "Greet someone",
    inputSchema: {
      type: "object",
      properties: { name: { type: "string" } },
      required: ["name"]
    }
  }]
}));

const transport = new StdioServerTransport();
await server.connect(transport);
```

---

## Ecosystem

### Official SDKs

| Language | Package | Status |
|---|---|---|
| Python | `mcp` (PyPI) | Stable |
| TypeScript / Node | `@modelcontextprotocol/sdk` | Stable |
| Java | `mcp-java` | Beta |
| Kotlin | `mcp-kotlin` | Beta |
| C# / .NET | `mcp-dotnet` | Beta |

### Hosts that support MCP natively

- Claude Desktop
- Claude Code (CLI)
- Cursor
- Zed
- Sourcegraph Cody
- Continue.dev

### Notable open-source servers

| Server | What it exposes |
|---|---|
| `mcp-server-github` | GitHub issues, PRs, repos |
| `mcp-server-slack` | Slack channels & messages |
| `mcp-server-postgres` | PostgreSQL query tool |
| `mcp-server-filesystem` | Local file read/write |
| `mcp-server-puppeteer` | Browser automation |
| `mcp-server-brave-search` | Web search |

---

## Roadmap

- [x] Core concepts & architecture docs
- [x] Quick-start examples (Python + TypeScript)
- [ ] Filesystem server walkthrough
- [ ] Database server with auth
- [ ] SSE (remote) server deployment guide
- [ ] Docker patterns for MCP servers
- [ ] Testing strategies for MCP servers
- [ ] Real-world case studies

---

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## Resources

- [Official Spec — modelcontextprotocol.io](https://modelcontextprotocol.io)
- [MCP GitHub Org](https://github.com/modelcontextprotocol)
- [Anthropic Blog: Introducing MCP](https://www.anthropic.com/news/model-context-protocol)
- [Python SDK Docs](https://github.com/modelcontextprotocol/python-sdk)
- [TypeScript SDK Docs](https://github.com/modelcontextprotocol/typescript-sdk)

---

## License

MIT — see [LICENSE](LICENSE) for details.

---

<p align="center">
  Built with curiosity by <a href="https://github.com/avinashmyerolkar">Avinash Yerolkar</a>
</p>
