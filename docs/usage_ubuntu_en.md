# Serial MCP Server Ubuntu / Linux User Guide (English)

For users who already installed the app: **GUI-first** (`serial_mcp_gui`). Command-line and environment variables are optional.

## 1. Before you start

- **MCP backend**: `serial_mcp_server` (where your package installed it—often `/usr/local/bin/serial_mcp_server`)
- **Recommended**: **`serial_mcp_gui`** (usually alongside the backend from the same distribution)
- Serial device available (for example `/dev/ttyUSB0`, `/dev/ttyACM0`)

## 2. Environment requirements and dependencies

### 2.1 Recommended environment

- Ubuntu 22.04 / 24.04 (or compatible distributions)
- x86_64 (amd64) architecture
- Network reachability (if you use remote HTTP mode)

### 2.2 Install required packages (recommended for local build/runtime)

```bash
sudo apt-get update
sudo apt-get install -y \
  cmake ninja-build g++ \
  qt6-base-dev libqt6serialport6-dev libqt6websockets6-dev
```

Notes:
- If you use a fully bundled release directory, you may not need all dev packages.
- If you build `serial_mcp_server` / `serial_mcp_gui` locally, install all packages above.

## 3. Serial access (usually required)

Add your user to `dialout`, then log in again:

```bash
sudo usermod -a -G dialout $USER
```

## 4. Recommended: use the GUI

1. **Start the GUI**  
   Run **`serial_mcp_gui`** from the terminal (full path if it is not on `PATH`) or from your application menu if the installer added one.

2. **Open “MCP Config && Claude Integration”**  
   - **Backend**: **C++** and the absolute path to **`serial_mcp_server`**.  
   - **Transport**: **stdio** or **http**.  
   - **WebSocket**: match the backend (default `127.0.0.1` / `8765`).  
   - **Serial (optional)**: e.g. `/dev/ttyUSB0` and baudrate; omit and use `serial_connect` in Claude.

3. **Connect Claude**  
   **Generate / copy** JSON into `.mcp.json` or your MCP settings; in **http** mode use the GUI integration. Optionally **save / load config.json**.

4. **Ordering**  
   Same as the [Windows guide](usage_windows_en.md): **stdio**—Claude starts the backend; **http**—start HTTP MCP in the GUI first.

### Serial tools in Claude

1. `serial_list_ports`  
2. `serial_connect` (for example `port=/dev/ttyUSB0`, `baudrate=115200`)  
3. `serial_write` / `serial_read`

## 5. Command line and environment (optional)

Replace paths below with the real location of **`serial_mcp_server`** on your system.

### 4.1 Hand-written MCP config (stdio)

```json
{
  "mcpServers": {
    "serial": {
      "command": "/usr/local/bin/serial_mcp_server",
      "args": [],
      "env": {
        "SERIAL_MCP_WS_HOST": "127.0.0.1",
        "SERIAL_MCP_WS_PORT": "8765"
      }
    }
  }
}
```

### 4.2 Auto-open serial at startup

```bash
export SERIAL_MCP_PORT=/dev/ttyUSB0
export SERIAL_MCP_BAUDRATE=115200
/usr/local/bin/serial_mcp_server
```

### 4.3 HTTP mode

```bash
SERIAL_MCP_TRANSPORT=http \
SERIAL_MCP_HTTP_HOST=0.0.0.0 \
SERIAL_MCP_HTTP_PORT=8443 \
SERIAL_MCP_HTTP_PATH=/mcp \
SERIAL_MCP_HTTP_TOKEN=your-token \
/usr/local/bin/serial_mcp_server
```

Example client on another machine:

```json
{
  "mcpServers": {
    "serial-remote": {
      "url": "http://ubuntu-hostname-or-ip:8443/mcp",
      "headers": {
        "Authorization": "Bearer your-token"
      }
    }
  }
}
```

## 6. Troubleshooting

- Health: `GET /healthz`  
- MCP path must be **`/mcp`**  
- If you use a token, Bearer must match exactly
