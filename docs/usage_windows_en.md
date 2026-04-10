# Serial MCP Server Windows User Guide (English)

For users who already installed the app: **GUI-first** (`serial_mcp_gui.exe`). Command-line and environment variables are optional for headless or scripted use.

## 1. Before you start

- **MCP backend**: `serial-mcp-cpp.exe` (where you installed or extracted it)
- **Recommended**: **`serial_mcp_gui.exe`** (settings, Claude MCP JSON, live serial traffic)
- Serial device plugged in (for example `COM3`, `COM6`)

## 2. Recommended: use the GUI

1. **Launch the GUI**  
   Run `serial_mcp_gui.exe`.

2. **Open “MCP Config && Claude Integration”**  
   - **Backend**: **C++** and the full path to **`serial-mcp-cpp.exe`**.  
   - **Transport**: **stdio** (Claude starts the backend locally) or **http** (this app hosts MCP; others connect by URL).  
   - **WebSocket**: must match the backend (default `127.0.0.1` / `8765`) for the traffic view and optional writes.  
   - **Serial (optional)**: fills auto-connect into the generated config; leave blank and use `serial_list_ports` / `serial_connect` in Claude.

3. **Connect Claude**  
   **Generate / copy** JSON and merge into your project `.mcp.json` or user MCP settings (follow Claude Code docs). In **http** mode you can use **add to Claude**. Optionally **save / load config.json**.

4. **stdio**  
   After Claude starts the backend, point the GUI at the same WebSocket endpoint.

5. **http**  
   **Start MCP (HTTP)** in the GUI first, then use the URL (and token if any) from the client.

### Serial tools in Claude

1. `serial_list_ports`  
2. `serial_connect` (for example `port=COM6`, `baudrate=115200`)  
3. `serial_write` / `serial_read`

## 3. Command line and environment (optional)

### 3.1 Hand-written MCP config (stdio)

Set `command` to the real path of **`serial-mcp-cpp.exe`** on your PC:

```json
{
  "mcpServers": {
    "serial": {
      "command": "C:\\Path\\To\\serial-mcp-cpp.exe",
      "args": [],
      "env": {
        "SERIAL_MCP_WS_HOST": "127.0.0.1",
        "SERIAL_MCP_WS_PORT": "8765"
      }
    }
  }
}
```

### 3.2 Auto-open serial when the backend starts

```powershell
$env:SERIAL_MCP_PORT="COM6"
$env:SERIAL_MCP_BAUDRATE="115200"
.\serial-mcp-cpp.exe
```

> `SERIAL_MCP_BAUDRATE` applies only if `SERIAL_MCP_PORT` is set.

### 3.3 HTTP mode

```powershell
$env:SERIAL_MCP_TRANSPORT="http"
$env:SERIAL_MCP_HTTP_HOST="0.0.0.0"
$env:SERIAL_MCP_HTTP_PORT="8443"
$env:SERIAL_MCP_HTTP_PATH="/mcp"
$env:SERIAL_MCP_HTTP_TOKEN="your-token"
.\serial-mcp-cpp.exe
```

Example client on another machine:

```json
{
  "mcpServers": {
    "serial-remote": {
      "url": "http://windows-hostname-or-ip:8443/mcp",
      "headers": {
        "Authorization": "Bearer your-token"
      }
    }
  }
}
```

## 4. Troubleshooting

- Health check: `GET /healthz` (same host/port as your HTTP settings)  
- MCP path must be **`/mcp`**, not `/healthz`  
- If you use a token, `Authorization: Bearer ...` must match exactly
