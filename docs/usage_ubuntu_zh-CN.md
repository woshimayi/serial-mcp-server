# Serial MCP Server Ubuntu / Linux 使用说明（中文）

面向已安装本软件的用户：以 **GUI（`serial_mcp_gui`）配置为主**，命令行与环境变量作为补充。

## 1. 运行前准备

- **MCP 后端**：`serial-mcp-cpp`（安装位置以你获得的软件为准，常见为 `/usr/local/bin/serial-mcp-cpp` 或安装包指定目录）
- **推荐**：**`serial_mcp_gui`**（与后端在同一安装包/目录中，或发布方单独提供）
- 串口设备已连接（如 `/dev/ttyUSB0`、`/dev/ttyACM0`）

## 2. 使用环境与依赖安装

### 2.1 建议使用环境

- Ubuntu 22.04 / 24.04（或兼容发行版）
- x86_64（amd64）架构
- 网络可访问（若使用 HTTP 远程模式）

### 2.2 安装必要依赖包（构建或本机运行推荐）

```bash
sudo apt-get update
sudo apt-get install -y \
  cmake ninja-build g++ \
  qt6-base-dev libqt6serialport6-dev libqt6websockets6-dev
```

## 3. 串口权限（通常需要）

当前用户加入 `dialout` 组后，重新登录生效：

```bash
sudo usermod -a -G dialout $USER
```

## 4. 推荐：用 GUI 完成配置

1. **启动 GUI**  
   在终端中运行安装包提供的 **`serial_mcp_gui`**（若桌面有快捷方式也可从菜单启动）。若命令不在默认路径，请使用完整路径，或先 `cd` 到程序所在目录。

2. **打开「MCP 配置与 Claude 集成」面板**  
   - **后端类型**：选 **C++**，填写 **`serial-mcp-cpp` 的绝对路径**。  
   - **传输**：**stdio** 或 **http**。  
   - **WebSocket**：与后端一致（默认 `127.0.0.1` / `8765`）。  
   - **串口（可选）**：如 `/dev/ttyUSB0` 与波特率；不填则在 Claude 里 `serial_connect`。

3. **接入 Claude**  
   **生成 / 复制** 配置并合并到 `.mcp.json` 或你的 Claude MCP 设置；**http** 模式可用界面提供的集成操作。可选 **保存 / 加载 config.json**。

4. **启动顺序**  
   与 [Windows 说明](usage_windows_zh-CN.md) 一致：**stdio** 由 Claude 启动后端；**http** 需先在 GUI 中启动 HTTP MCP。

### 在 Claude 里操作串口

1. `serial_list_ports`  
2. `serial_connect`（例如 `port=/dev/ttyUSB0`、`baudrate=115200`）  
3. `serial_write` / `serial_read`

## 5. 命令行与环境变量（辅助）

以下路径请换成你机器上 **`serial-mcp-cpp` 的实际位置**。

### 4.1 手写 MCP 配置（stdio）

```json
{
  "mcpServers": {
    "serial": {
      "command": "/usr/local/bin/serial-mcp-cpp",
      "args": [],
      "env": {
        "SERIAL_MCP_WS_HOST": "127.0.0.1",
        "SERIAL_MCP_WS_PORT": "8765"
      }
    }
  }
}
```

### 4.2 启动时自动连接串口

```bash
export SERIAL_MCP_PORT=/dev/ttyUSB0
export SERIAL_MCP_BAUDRATE=115200
/usr/local/bin/serial-mcp-cpp
```

### 4.3 HTTP 模式

```bash
SERIAL_MCP_TRANSPORT=http \
SERIAL_MCP_HTTP_HOST=0.0.0.0 \
SERIAL_MCP_HTTP_PORT=8443 \
SERIAL_MCP_HTTP_PATH=/mcp \
SERIAL_MCP_HTTP_TOKEN=your-token \
/usr/local/bin/serial-mcp-cpp
```

其他机器上的客户端示例：

```json
{
  "mcpServers": {
    "serial-remote": {
      "url": "http://ubuntu-主机名或IP:8443/mcp",
      "headers": {
        "Authorization": "Bearer your-token"
      }
    }
  }
}
```

## 6. 排错要点

- 健康检查：`GET /healthz`  
- MCP 路径必须是 **`/mcp`**  
- 若启用令牌，客户端 Bearer 必须与服务器一致
