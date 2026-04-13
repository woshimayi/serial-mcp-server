# Serial MCP Server Windows 使用说明（中文）

面向已安装本软件的用户：以 **GUI（`serial_mcp_gui.exe`）配置为主**，命令行与环境变量在无界面或脚本场景下作为补充。

## 1. 运行前准备

- **MCP 后端**：`serial_mcp_server.exe`（安装目录或你解压/放置的位置）
- **推荐**：**`serial_mcp_gui.exe`**（配置参数、生成 Claude MCP 配置、查看串口收发）
- 串口已连接（常见端口名如 `COM3`、`COM6`）

## 2. 推荐：用 GUI 完成配置

1. **启动 GUI**  
   双击或运行 `serial_mcp_gui.exe`。

2. **打开「MCP 配置与 Claude 集成」面板**  
   - **后端类型**：选 **C++**，在浏览框中选择 **`serial_mcp_server.exe` 的完整路径**。  
   - **传输**：  
     - **stdio**：由 Claude Code 直接启动后端（本机常用）。  
     - **http**：由本程序托管 HTTP MCP，其他设备用 URL 连接。  
   - **WebSocket**：主机与端口须与后端一致（默认 `127.0.0.1` / `8765`），用于在界面里查看收发，也可通过 WebSocket 向串口写入。  
   - **串口（可选）**：填写端口与波特率后，生成的配置会带上自动连接参数；不填则可在 Claude 里用 `serial_list_ports` / `serial_connect`。

3. **接入 Claude**  
   - 使用 **生成 / 复制 Claude 配置**，把 JSON 合并到项目里的 `.mcp.json` 或用户配置里的 `mcpServers`（具体以 Claude Code 文档为准）。  
   - **http** 模式下可使用界面提供的 **添加到 Claude**。  
   - 可选：**保存 config.json** / **加载**，方便下次恢复。

4. **stdio 模式**  
   Claude 启动后端后，让 GUI 连到相同的 WebSocket 地址即可看到流量。

5. **http 模式**  
   先在 GUI 里 **启动 MCP（HTTP）**，再在客户端使用界面里给出的地址与 Token（若已设置）。

### 在 Claude 里操作串口

1. `serial_list_ports`  
2. `serial_connect`（例如 `port=COM6`、`baudrate=115200`）  
3. `serial_write` / `serial_read`

## 3. 命令行与环境变量（辅助）

适用于不打开 GUI、或用计划任务/脚本启动后端的场景。

### 3.1 手写 MCP 配置（stdio）

将下面 `command` 改成你本机 **`serial_mcp_server.exe` 的实际路径**：

```json
{
  "mcpServers": {
    "serial": {
      "command": "C:\\路径\\serial_mcp_server.exe",
      "args": [],
      "env": {
        "SERIAL_MCP_WS_HOST": "127.0.0.1",
        "SERIAL_MCP_WS_PORT": "8765"
      }
    }
  }
}
```

### 3.2 启动时自动连接串口

在 PowerShell 中（先 `cd` 到后端所在目录，或写全路径）：

```powershell
$env:SERIAL_MCP_PORT="COM6"
$env:SERIAL_MCP_BAUDRATE="115200"
.\serial_mcp_server.exe
```

> 只有设置了 `SERIAL_MCP_PORT` 时，`SERIAL_MCP_BAUDRATE` 才会生效。

### 3.3 HTTP 模式（远程 URL）

```powershell
$env:SERIAL_MCP_TRANSPORT="http"
$env:SERIAL_MCP_HTTP_HOST="0.0.0.0"
$env:SERIAL_MCP_HTTP_PORT="8443"
$env:SERIAL_MCP_HTTP_PATH="/mcp"
$env:SERIAL_MCP_HTTP_TOKEN="your-token"
.\serial_mcp_server.exe
```

其他电脑上的客户端示例（把主机名和 Token 换成你的）：

```json
{
  "mcpServers": {
    "serial-remote": {
      "url": "http://windows-主机名或IP:8443/mcp",
      "headers": {
        "Authorization": "Bearer your-token"
      }
    }
  }
}
```

## 4. 排错要点

- 健康检查：`GET /healthz`（浏览器或 `curl` 访问时，地址和端口与你在 http 模式里设置的一致）  
- MCP 地址路径必须是 **`/mcp`**，不要填成 `/healthz`  
- 若设置了访问令牌，客户端里的 `Authorization: Bearer ...` 必须与服务器完全一致
