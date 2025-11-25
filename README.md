

# Weibo MCP Server

获取前 N 条微博热搜的 MCP 服务，基于 `fastmcp` 构建，内置 **Stdio、本地 SSE、Streamable HTTP** 三种运行脚本，适配本地和远程多端场景。

---

## 🚀 快速开始

```bash
git clone https://github.com/bossdong955/weibo-mcp-server.git
cd weibo-mcp-server

# 创建并进入 Conda 环境
conda create -n weibo-mcp-server python=3.11
conda activate weibo-mcp-server

# 安装依赖（注意：请确认本机已卸载旧版 mcp 包）
pip install -r requirements.txt
```

`requirements.txt` 默认包含：

- fastmcp
- httpx
- uvicorn
- starlette

---

## 🧭 运行模式

| 模式 | 脚本 | 典型场景 | 监听方式 |
| --- | --- | --- | --- |
| Stdio | `resou_stdio.py` | Claude Desktop / VS Code Cline 本地插件 | 标准输入输出 |
| SSE | `resou_sse.py` | 部署在服务器，通过 SSE 远程连接 | `http://HOST:PORT/sse` |
| Streamable HTTP | `resou_streamable_http.py` | 支持 HTTP 长连接的客户端 | `http://HOST:PORT/mcp` |

启动任意脚本后都可以调用 `hot_search(n=20)` 获取热搜，`n` 为返回条数上限。

---

### 1. Stdio（本地推荐）

```bash
python resou_stdio.py
```

VS Code Cline / Claude Desktop `mcpServers` 示例（将路径替换成你本机的脚本路径）：

```json
{
  "mcpServers": {
    "weiboresou": {
      "command": "conda",
      "args": [
        "run",
        "-n",
        "weibo-mcp-server",
        "--no-capture-output",
        "python",
        "E:/your/path/resou_stdio.py"
      ],
      "env": {
        "PYTHONUTF8": "1"
      }
    }
  }
}
```

> 已在脚本中调用 `mcp.run(transport="stdio")`，无需再使用 `mcp run ...`。

---

### 2. SSE（远程流式）

```bash
python resou_sse.py --host 0.0.0.0 --port 8006
```

在 VS Code Cline -> `Remote Servers` 中设置：

- URL: `http://<host>:8006/sse`
- 可选 Header：`Authorization` 等自定义认证信息（如有需要）

---

### 3. Streamable HTTP

```bash
python resou_streamable_http.py --host 0.0.0.0 --port 8005
```

远程客户端配置示例：

- URL: `http://<host>:8005/mcp`
- 该模式返回可流式消费的 HTTP 响应，适合自定义前端或其他 MCP 客户端。

---

## 🛠️ 常见问题

1. **编码报错（GBK 等）**：在客户端配置中加入 `PYTHONUTF8=1`。
2. **依赖冲突**：如果之前安装过 `mcp` 包，请先卸载 `pip uninstall mcp`，再安装 `fastmcp`。
3. **网络超时**：`hot_search` 默认 10s 超时。可在脚本中调整 `httpx.AsyncClient` 的 `timeout` 参数。

---

## ✅ 近期改动

- 切换至独立的 `fastmcp` 包，移除对旧 `mcp` CLI 的依赖。
- 为不同传输方式提供了独立脚本（stdio / sse / streamable-http），启动方式更直观。
- 更新示例配置，默认通过 `python your_script.py` 即可运行。