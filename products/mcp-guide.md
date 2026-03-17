# MCP Server Integration Guide — Model Context Protocol for AI Agents

**Version:** 1.0 | **Level:** Advanced | **Setup Time:** 45-60 minutes

Connect any tool to your AI agent via Model Context Protocol. MuninnDB, Notion, Postgres, custom APIs — MCP makes them all agent-accessible. Includes 5 working server configs and a custom server template.

---

## What's In This Package

- MCP protocol overview
- MuninnDB MCP setup
- OpenClaw MCP configuration
- 5 MCP server examples
- Custom MCP server template (Python)
- Security considerations
- Multi-server routing

---

## What is MCP?

Model Context Protocol (MCP) is an open standard that lets AI models interact with external tools and data sources through a standardized interface.

**Why it matters:**
- One protocol, any tool — build once, use with Claude, GPT, Gemini
- Structured tool definitions — the model knows exactly what's available and how to call it
- Secure — tools run in separate processes, not in the model's context
- Composable — chain multiple MCP servers for rich agent capabilities

**Architecture:**
```
AI Model (Claude) 
    ↓ sends tool calls
MCP Client (OpenClaw)
    ↓ routes to appropriate server
MCP Server (your tool)
    ↓ executes the action
Returns structured result
    ↑ back to model
```

---

## MCP Protocol Basics

### Tool Definition Format

Every MCP tool has:
```json
{
  "name": "tool_name",
  "description": "What this tool does — be specific",
  "inputSchema": {
    "type": "object",
    "properties": {
      "param1": {
        "type": "string",
        "description": "What this parameter is"
      }
    },
    "required": ["param1"]
  }
}
```

### Server Communication

MCP servers communicate via stdio (standard input/output) or HTTP:

- **Stdio:** server runs as a child process, communicates via stdin/stdout
- **HTTP/SSE:** server runs as a service, communicates via HTTP + Server-Sent Events

Most local tools use stdio. Remote/cloud tools use HTTP.

---

## OpenClaw MCP Configuration

Add MCP servers to your OpenClaw config:

```json
{
  "mcp": {
    "servers": {
      "muninndb": {
        "type": "stdio",
        "command": "node",
        "args": ["/path/to/muninndb-mcp/index.js"],
        "env": {
          "MUNINN_DB_PATH": "/path/to/your/muninn.db"
        }
      },
      "filesystem": {
        "type": "stdio",
        "command": "npx",
        "args": ["@modelcontextprotocol/server-filesystem", "/your/workspace/path"]
      },
      "postgres": {
        "type": "stdio",
        "command": "npx",
        "args": ["@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"]
      }
    }
  }
}
```

---

## Example 1: MuninnDB (Agent Memory)

MuninnDB is a local key-value and semantic search database designed for AI agents.

### Install

```bash
npm install -g muninndb
# or clone and run locally
git clone https://github.com/[muninndb-repo]
cd muninndb-mcp && npm install
```

### Configure

```json
{
  "mcp": {
    "servers": {
      "memory": {
        "type": "stdio",
        "command": "node",
        "args": ["/usr/local/lib/node_modules/muninndb/server.js"],
        "env": {
          "DB_PATH": "/Users/yourname/.openclaw/memory/muninn.db"
        }
      }
    }
  }
}
```

### Available Tools

Once configured, your agent gets:
- `memory_store(key, value)` — store any data
- `memory_retrieve(key)` — get by exact key
- `memory_search(query)` — semantic search across stored content
- `memory_list()` — list all keys
- `memory_delete(key)` — remove an entry

### Usage Example

```
Store this research finding in memory:

Key: "gpt4o-pricing-update-2024-01"
Value: "GPT-4o pricing dropped to $0.005/1K input tokens on Jan 15 2024. 50% reduction. Context: OpenAI competitive pressure from Claude 3."

Use the memory MCP tool to store this.
```

---

## Example 2: Filesystem Access

The official MCP filesystem server lets your agent read and write files:

### Install

```bash
npm install -g @modelcontextprotocol/server-filesystem
```

### Configure

```json
{
  "mcp": {
    "servers": {
      "filesystem": {
        "type": "stdio",
        "command": "npx",
        "args": [
          "@modelcontextprotocol/server-filesystem",
          "/Users/yourname/.openclaw/workspace",
          "/Users/yourname/Documents/Projects"
        ]
      }
    }
  }
}
```

### Available Tools

- `read_file(path)` — read file contents
- `write_file(path, content)` — write to file
- `list_directory(path)` — list files
- `create_directory(path)` — create folder
- `move_file(source, destination)` — move/rename
- `search_files(pattern)` — find files by pattern

---

## Example 3: PostgreSQL Database

### Install

```bash
npm install -g @modelcontextprotocol/server-postgres
```

### Configure

```json
{
  "mcp": {
    "servers": {
      "postgres": {
        "type": "stdio",
        "command": "npx",
        "args": [
          "@modelcontextprotocol/server-postgres",
          "postgresql://username:password@localhost:5432/dbname"
        ]
      }
    }
  }
}
```

### Available Tools

- `query(sql)` — execute a SELECT query
- `execute(sql)` — execute INSERT/UPDATE/DELETE
- `list_tables()` — show all tables
- `describe_table(name)` — get table schema

---

## Example 4: SQLite (Local Database)

```bash
npm install -g @modelcontextprotocol/server-sqlite
```

### Configure

```json
{
  "mcp": {
    "servers": {
      "sqlite": {
        "type": "stdio",
        "command": "npx",
        "args": [
          "@modelcontextprotocol/server-sqlite",
          "--db-path", "/path/to/your/database.db"
        ]
      }
    }
  }
}
```

---

## Example 5: GitHub MCP

```bash
npm install -g @modelcontextprotocol/server-github
```

### Configure

```json
{
  "mcp": {
    "servers": {
      "github": {
        "type": "stdio",
        "command": "npx",
        "args": ["@modelcontextprotocol/server-github"],
        "env": {
          "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_yourtoken"
        }
      }
    }
  }
}
```

### Available Tools

- `create_issue(owner, repo, title, body)` — open issues
- `create_pull_request(...)` — create PRs
- `get_file_contents(owner, repo, path)` — read files from repos
- `push_files(...)` — write files to repos
- `search_repositories(query)` — search GitHub

---

## Custom MCP Server Template (Python)

Build a custom server to connect any tool:

### Install MCP SDK

```bash
pip install mcp
```

### Server Template

Create `my-server/server.py`:

```python
#!/usr/bin/env python3
"""Custom MCP Server — replace with your tool's functionality."""

import asyncio
import json
from mcp.server import Server
from mcp.server.models import InitializationOptions
from mcp.server.stdio import stdio_server
from mcp.types import (
    CallToolRequestParams,
    CallToolResult,
    ListToolsResult,
    TextContent,
    Tool,
)

# Initialize the server
server = Server("my-custom-server")

# Define your tools
@server.list_tools()
async def list_tools() -> ListToolsResult:
    return ListToolsResult(
        tools=[
            Tool(
                name="my_tool",
                description="Description of what this tool does",
                inputSchema={
                    "type": "object",
                    "properties": {
                        "param1": {
                            "type": "string",
                            "description": "Description of param1",
                        },
                        "param2": {
                            "type": "integer",
                            "description": "Description of param2",
                        },
                    },
                    "required": ["param1"],
                },
            ),
        ]
    )

# Handle tool calls
@server.call_tool()
async def call_tool(name: str, arguments: dict) -> CallToolResult:
    if name == "my_tool":
        # Your tool logic here
        param1 = arguments["param1"]
        param2 = arguments.get("param2", 0)
        
        # Do the work
        result = f"Processed {param1} with {param2}"
        
        return CallToolResult(
            content=[TextContent(type="text", text=result)]
        )
    else:
        raise ValueError(f"Unknown tool: {name}")

# Run the server
async def main():
    async with stdio_server() as (read_stream, write_stream):
        await server.run(
            read_stream,
            write_stream,
            InitializationOptions(
                server_name="my-custom-server",
                server_version="0.1.0",
            ),
        )

if __name__ == "__main__":
    asyncio.run(main())
```

### Register Your Custom Server

```json
{
  "mcp": {
    "servers": {
      "my-server": {
        "type": "stdio",
        "command": "python3",
        "args": ["/path/to/my-server/server.py"],
        "env": {
          "MY_API_KEY": "your-key"
        }
      }
    }
  }
}
```

---

## Multi-Server Routing

When you have multiple MCP servers, your agent routes automatically based on tool names.

Example of an agent session with multiple servers:

```
Available tools from MCP servers:
- memory_store, memory_retrieve, memory_search (from: muninndb)
- read_file, write_file, list_directory (from: filesystem)
- query, execute (from: postgres)
- create_issue, get_file_contents (from: github)

The agent automatically selects the right server for each tool call.
```

### Server Organization Tip

Name tools with server prefixes to avoid collisions:
- `muninn_store`, `muninn_search`
- `fs_read`, `fs_write`
- `db_query`, `db_execute`
- `gh_create_issue`, `gh_get_file`

---

## Security Considerations

### What MCP servers can access

- Filesystem server: only paths you explicitly whitelist in the config
- Database servers: the connection string you provide (consider read-only user)
- Custom servers: whatever your code does

### Best Practices

```markdown
1. **Least privilege** — give each server only the access it needs
   - DB server: create a read-only database user for query-only servers
   - Filesystem: only whitelist specific directories, not home root

2. **Secrets in env, not args** — API keys go in `env`, not in `args` array

3. **Audit what your custom servers do** — if it touches external APIs, log it

4. **Don't run untrusted MCP servers** — treat them like code you're installing

5. **Network isolation** — local servers should only listen on localhost

6. **Token rotation** — rotate MCP server API tokens same as any other secret
```

---

## Troubleshooting

**Server not starting**
- Test the command manually: `node /path/to/server.js`
- Check error output: `node /path/to/server.js 2>&1`
- Verify all dependencies are installed: `npm list` in the server directory

**Tools not appearing**
- Restart OpenClaw after config changes
- Check that the `mcp.servers` config is valid JSON (no trailing commas)
- Verify the server name in config matches what you're referencing

**Server crashes on tool call**
- Test with a simple input first
- Add error handling to custom servers
- Check the server's stderr output for the error

**Slow tool calls**
- MCP stdio servers have process startup overhead
- Consider HTTP servers for frequently-used tools
- Cache results in MuninnDB to reduce repeated external calls

---

*Built on OpenClaw. MCP is an open standard — see modelcontextprotocol.io for full spec.*
