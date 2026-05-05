# CLAUDE.md

This file describes the project structure and conventions for working in this repository with Claude Code.

## Project

**mcp-playbook** is a structured reference and learning resource for Model Context Protocol (MCP). It targets developers who want to understand, build, and deploy MCP servers and clients. The repo is also designed to be readable as a portfolio piece.

## Directory Conventions

| Directory | Purpose |
|---|---|
| `01-concepts/` | Theory, architecture docs, no runnable code |
| `02-servers/` | Runnable MCP server examples |
| `03-clients/` | Client connection examples |
| `04-tools/` | Tool definition patterns |
| `05-resources/` | Resource exposure patterns |
| `06-prompts/` | Prompt template patterns |
| `07-real-world/` | Production patterns, error handling, auth |
| `08-integrations/` | Third-party integration recipes |
| `09-recipes/` | Copy-paste snippets for common use cases |

## Code Conventions

- Python examples use the `mcp` PyPI package
- TypeScript examples use `@modelcontextprotocol/sdk`
- Each runnable example must have its own `README.md` with setup instructions
- Keep examples minimal and focused — one concept per file
- No placeholder TODOs in committed code

## Writing Conventions

- Docs are written in GitHub-flavored Markdown
- Diagrams use ASCII art (no external image dependencies)
- Every code block must specify a language tag
- Prefer tables over bullet lists for structured comparisons

## What NOT to add

- Framework-specific wrappers (LangChain, LlamaIndex) unless the section is explicitly about them
- Locked dependencies — keep `requirements.txt` / `package.json` minimal
- AI-generated filler text — every sentence should earn its place
