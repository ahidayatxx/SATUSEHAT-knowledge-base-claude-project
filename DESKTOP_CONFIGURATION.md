# Claude Desktop Configuration

## MCP Server Configuration
Tambahkan konfigurasi berikut ke file `claude_desktop_config.json`:

- [Tavily MCP](https://github.com/tavily-ai/tavily-mcp)
- [Brave MCP](https://github.com/brave/brave-search-mcp-server)

```json
{
  "mcpServers": {
    "satusehat-tavily": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-tavily"
      ],
      "env": {
        "TAVILY_API_KEY": "tvly-YOUR_TAVILY_API_KEY_HERE"
      }
    },
    "satusehat-brave": {
      "command": "npx", 
      "args": [
        "-y",
        "@modelcontextprotocol/server-brave-search"
      ],
      "env": {
        "BRAVE_API_KEY": "YOUR_BRAVE_API_KEY_HERE"
      }
    }
  }
}
```
