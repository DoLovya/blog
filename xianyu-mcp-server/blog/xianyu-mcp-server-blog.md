# 让 AI 帮我管闲鱼店铺：xianyu-mcp-server 体验分享

> 基于 MCP 协议把闲鱼商品、会话、消息等能力封装为标准工具，接入 Trae、Claude Desktop、Cursor 等 AI 客户端即可调用
>
> 📦 **项目源码**：[github.com/DoLovya/xianyu-mcp-server](https://github.com/DoLovya/xianyu-mcp-server)

## 〇、项目背景（为什么做这个）

在闲鱼上卖东西久了，每天重复的操作很耗精力：

1. **商品管理重复且琐碎**：上架、改价、下架、重新上架，每个动作都要在手机 App 里反复点。想批量调整价格或下架低价商品时，没有现成的批量操作入口，只能逐个手动处理。
2. **买家消息回复不及时**：白天上班时手机不在手边，买家咨询得不到及时回复，经常错过成交机会。常见的"能便宜点吗""还在吗"这类重复问题，如果能用 AI 先顶一轮回复就好了。

既然这些操作本质上都是调接口，为什么不能让 AI 直接来干？

于是用 **Python + pyxianyu + MCP 协议** 的方案：把闲鱼的登录、商品、会话、消息等 HTTP API 能力封装成 20 个标准 MCP 工具，接入 Trae、Cursor、Claude Desktop 等支持 MCP 的 AI 客户端后，直接用自然语言就能让 AI 调用。项目内置风控护栏（读写分离限速、写操作串行化、指数退避、强风控冷却），避免自动化触发平台封号。

现在，**批量改价、下架商品、回复买家消息**，说一句话 AI 就能做完，不用再在 App 里逐个点。

## 一、项目简介

xianyu-mcp-server 是一个基于 pyxianyu 封装的闲鱼 MCP 服务，将闲鱼商品管理、会话消息、扫码登录等能力封装为标准 MCP 工具。任何支持 MCP 协议的 AI 客户端（Trae、Claude Desktop、Cursor、VS Code、Cherry Studio）接入后，均可通过自然语言直接调用闲鱼能力。

### 核心特性

- 🔧 **20 个 MCP 工具**：覆盖登录、商品、会话、消息、媒体上传、扫码登录六大场景
- 🤖 **5 大客户端接入**：Trae / Claude Desktop / Cursor / VS Code / Cherry Studio
- 📱 **扫码登录**：完整二维码登录链路，首次使用无需手动抓 Cookie
- 🛡️ **风控护栏**：读写分离限速 + 写操作串行化 + 指数退避 + 强风控冷却
- 📦 **双运行模式**：`uv run`（推荐）和 `pip install`（替代）两条路径
- 🔄 **stdio / HTTP 双传输**：默认 stdio，Cherry Studio 等可接 HTTP 模式

---

## 二、系统架构

<img src="https://cdn.jsdelivr.net/gh/DoLovya/blog@main/xianyu-mcp-server/blog/images/architecture-diagram.jpg" alt="系统架构图" width="70%">

### 数据流说明

1. **AI 客户端 → MCP 服务**：AI 客户端（Trae/Cursor/Claude）通过 stdio 或 HTTP 连接到 xianyu-mcp-server
2. **MCP 服务 → pyxianyu**：工具调用委托给 pyxianyu 底层库，执行 HTTP API 请求
3. **pyxianyu → 闲鱼 Web API**：通过逆向分析的 HTTP 接口和 WebSocket 连接与闲鱼服务端通信
4. **风控护栏层**：所有请求经过 RequestGuardrails，读写操作分别限速、串行化写操作、异常时退避重试

### 为什么用 MCP 协议？

MCP（Model Context Protocol）是标准化的 AI 工具调用协议：

- AI 客户端只需配置一次，即可在对话中调用所有注册的 MCP 工具
- 不需要为每个 AI 产品单独做适配
- 支持 stdio 和 HTTP 两种传输模式

---

## 三、技术选型

项目在架构层面有几个关键选型决策：

- **MCP 协议而非 REST API**：REST API 需要为每个 AI 产品单独做适配，而 MCP 是标准协议，一次封装即可接入所有支持的客户端。闲鱼能力以"工具"形式暴露，AI 自动根据上下文决定调哪个。
- **Python 而非 Node.js**：pyxianyu 已有成熟的 Python 实现，逆向分析社区也以 Python 为主，Cookie 签名、WebSocket 协议等都有现成参考。
- **uv 而非 pip**：uv 依赖安装速度更快，CI 缓存友好，同时保留了 pip 作为后备路径。
- **FastMCP SDK**：MCP 官方 Python SDK，用 `@mcp.tool` 装饰器即可注册工具，不需要手写协议层代码。

---

## 四、快速开始

### 4.1 克隆仓库

```bash
git clone https://github.com/DoLovya/xianyu-mcp-server.git
cd xianyu-mcp-server
git submodule update --init --recursive
cp .env.example .env
```

> 💡 `.env` 先留空也行，项目会引导扫码登录获取 Cookie。

### 4.2 安装依赖

机器上能装 `uv` 的话，优先用 `uv`：

```bash
uv pip install -e third_party/pyxianyu
uv pip install -e .
```

装不上 `uv` 也没关系，直接退回 `pip`：

```bash
pip install -e third_party/pyxianyu
pip install -e .
```

### 4.3 启动 MCP 服务

默认走 `stdio`：

```bash
# 推荐（需要 uv）
uv run xianyu-mcp

# 或使用 pip 方案
python -m xianyu_mcp.server
```

如需 HTTP 模式：

```bash
# 推荐（需要 uv）
uv run xianyu-mcp --http

# 或使用 pip 方案
python -m xianyu_mcp.server --http
```

HTTP 模式默认监听：

```text
http://localhost:8000/mcp
```

> ⚠️ **不要使用 `uvx xianyu-mcp`**：本项目未发布到 PyPI，`uvx` 会安装到同名第三方包并报错（如 `AttributeError: 'Server' object has no attribute 'list_tools'`）。

### 4.4 客户端接入

Trae 项目级配置示例（已内置在仓库 `.trae/mcp.json`）：

```json
{
  "mcpServers": {
    "xianyu-mcp-server": {
      "command": "uv",
      "args": ["--directory", "${workspaceFolder}", "run", "xianyu-mcp"]
    }
  }
}
```

不支持 `${workspaceFolder}` 的客户端（如 Claude Desktop 全局配置），请使用绝对路径：

```json
{
  "mcpServers": {
    "xianyu-mcp-server": {
      "command": "uv",
      "args": [
        "--directory",
        "C:\\Users\\<user>\\Code\\xianyu-mcp-server",
        "run",
        "xianyu-mcp"
      ]
    }
  }
}
```

装不上 `uv` 时退回 Python 方式：

```json
{
  "mcpServers": {
    "xianyu-mcp-server": {
      "command": "python",
      "args": ["-m", "xianyu_mcp.server"],
      "cwd": "${workspaceFolder}"
    }
  }
}
```

| 客户端 | 配置文件 | 支持 `${workspaceFolder}` | 备注 |
|--------|----------|--------------------------|------|
| **Trae** | `.trae/mcp.json` | 是 | 配置后重载工作区 |
| **Claude Desktop** | `claude_desktop_config.json` | 否，需绝对路径 | 保存后重启 |
| **Cursor** | `.cursor/mcp.json` | 项目级支持 | 全局配置需绝对路径 |
| **VS Code** | `.vscode/mcp.json` | 是 | 需 VS Code 1.102+ |
| **Cherry Studio** | UI 配置 | N/A | 可选 stdio 或 HTTP |

---

## 五、功能详解

当前 xianyu-mcp-server 已开放 **20 个 MCP 工具**，分成 5 组。每张表同时列出工具的参数与说明。

### 5.1 登录与登录态维护

| 工具 | 参数 | 只读 | 说明 |
|------|------|------|------|
| `validate_login` | 无 | ✅ | 校验当前 Cookie 是否有效，并尝试换取 accessToken |
| `refresh_login` | 无 | — | 刷新当前登录态 |
| `qr_login_generate` | 无 | — | 生成扫码登录二维码（返回 session_id 与 base64 data-url） |
| `qr_login_status` | `session_id: str` | ✅ | 查询扫码登录状态，含人脸验证二维码 data-url |
| `qr_login_cookie` | `session_id: str` | ✅ | 扫码成功后获取完整 Cookie |
| `qr_login_save_env` | `session_id: str, env_path: str = ".env"` | — | 将扫码得到的 Cookie 直接写回 `.env` |

首次使用无需手动抓 Cookie：`.env` 留空启动，项目引导扫码，成功后可直接写回环境文件。如遇人脸验证或额外风控，工具会返回状态信息。

### 5.2 商品搜索与店铺管理

| 工具 | 参数 | 只读 | 说明 |
|------|------|------|------|
| `search_items` | `keyword, page_number=1, rows_per_page=20, sort_field="", sort_value=""` | ✅ | 按关键词搜索闲鱼商品，支持分页与排序 |
| `get_item_detail` | `item_id: str` | ✅ | 获取指定商品详情 |
| `get_item_edit_detail` | `item_id: str` | ✅ | 获取商品在 PC 编辑页的详情 |
| `list_my_items` | `page_size: int = 20` | ✅ | 自动翻页拉取当前账号全部商品 |
| `downshelf_item` | `item_id: str` | — | 下架指定商品 |
| `reshelf_item` | `item_id: str, source_id: str = ""` | — | 通过 PC 编辑重发布链路重新上架商品 |
| `edit_item` | `item_id: str, payload: dict = None, overrides: dict = None` | — | 编辑商品标题、价格、描述等信息 |
| `publish_physical_item` | `title: str, price: str, desc: str, images: list[str]` | — | 发布新实体商品，支持自动上传图片 |

### 5.3 用户信息与会话消息

| 工具 | 参数 | 只读 | 说明 |
|------|------|------|------|
| `get_my_profile` | 无 | ✅ | 获取当前登录账号个人信息 |
| `list_conversations` | `max_items=1000, include_hidden=False, only_top=False` | ✅ | 拉取最近会话列表 |
| `list_conversation_messages` | `cid: str, max_items=50` | ✅ | 获取指定会话历史消息 |
| `send_text_message` | `to_user_id: str, item_id: str, text: str` | — | 发送文本消息 |
| `send_image_message` | `to_user_id: str, item_id: str, image: str` | — | 发送图片消息 |

### 5.4 媒体上传

| 工具 | 参数 | 只读 | 说明 |
|------|------|------|------|
| `upload_media` | `media: str` | — | 上传本地文件或 URL 素材，返回可复用媒体链接 |

媒体上传能力既给 `publish_physical_item` 用，也可用于图片消息和商品编辑复用。

### 5.5 风控护栏

风控护栏不是单独的 MCP 工具，而是内置在所有请求的调用链路中（源码：`src/xianyu_mcp/guardrails.py`）：

| 机制 | 说明 | 默认参数 |
|------|------|----------|
| 读写分离限速 | 读操作和写操作使用不同节奏 | 读 1.2s 间隔，写 20s 间隔 |
| 写操作串行化 | 通过 `_write_lock` 保证写请求串行执行 | — |
| 指数退避 | 遇到可疑错误自动退避重试 | 基准 2s，上限 120s |
| 强风控冷却 | 识别 `FAIL_SYS_USER_VALIDATE` 后进入冷却 | 冷却 1800s（30 分钟） |

> 💡 所有参数均通过环境变量可调，如 `XIANYU_GUARD_READ_MIN_INTERVAL`、`XIANYU_GUARD_WRITE_MIN_INTERVAL` 等。

---

## 六、部署上线

### 6.1 环境要求

- Python 3.11+
- `uv`（或使用 `pip` 替代）
- 闲鱼登录后的完整 Cookie（可手动抓取，或先启动 MCP 后使用 `qr_login_*` 工具扫码获取）

### 6.2 CI/CD

项目使用 GitHub Actions 持续集成，配置文件为 `.github/workflows/ci.yml`：

| 工作流 | 触发条件 | 说明 |
|--------|----------|------|
| `ci.yml` → test | PR、push to main/master | Python 3.11/3.12/3.13 矩阵测试 + 安装模式导入验证 |
| `ci.yml` → build | test 通过后 | 构建 wheel 和 sdist 并上传 artifact |

**CI 关键步骤**：

1. Checkout（含 submodule）
2. 安装 Python + uv（带缓存）
3. `uv pip install -e .` 安装依赖
4. `python -m unittest discover -s tests` 运行测试
5. 删除 submodule 后验证 import（确保 PyPI 安装模式可用）

### 6.3 Cookie 配置

在 `.env` 中填写登录态（二选一）：

```ini
# 方式一：直接写入完整 Cookie
XIANYU_COOKIE=你的完整闲鱼 Cookie

# 方式二：Cookie 存放在单独文件中
XIANYU_COOKIE_FILE=./cookie.txt
```

优先级：配置了 `XIANYU_COOKIE` 时，`XIANYU_COOKIE_FILE` 会被忽略。每次工具调用前会重新读取 `.env`，修改 Cookie 后无需重启服务。

### 6.4 首次配置模式

`.env` 为空时启动 MCP，服务会自动进入"首次配置模式"：

1. 在本机 `127.0.0.1` 打开网页展示二维码
2. 手机闲鱼/淘宝扫码确认
3. 扫码成功后自动把 Cookie 写回 `.env`

相关开关（可选）：

```ini
XIANYU_SETUP_ENABLED=1        # 0 表示禁用首次配置模式
XIANYU_SETUP_AUTOSTART=1      # 0 表示启动时不自动弹出
XIANYU_SETUP_AUTO_OPEN=1      # 0 表示不自动打开浏览器
XIANYU_SETUP_AUTO_WRITE_ENV=1 # 0 表示不自动写入 .env
```

---

## 七、技术栈与选型

| 技术 | 版本 | 选型理由 |
|------|------|----------|
| Python | >=3.11 | pyxianyu 已有成熟的 Python 实现，逆向分析社区以 Python 为主 |
| mcp[cli] | >=1.0.0,<2.0.0 | MCP 官方 Python SDK，`@mcp.tool` 装饰器注册工具，无需手写协议层 |
| pyxianyu | >=0.1.0 | 闲鱼 HTTP/WebSocket 能力底层库，已封装签名、Cookie 管理等 |
| python-dotenv | >=1.0.0 | 环境变量管理，支持运行时热加载 |
| requests | >=2.31.0 | 同步 HTTP 请求 |
| httpx | >=0.27.0 | 异步 HTTP 客户端 |
| loguru | >=0.7.2 | 轻量日志库，支持文件轮转和异步写入 |
| websockets | >=12.0 | WebSocket 通信（消息监听等） |
| msgpack | >=1.0.0 | 消息序列化 |
| blackboxprotobuf | >=1.0.1 | Protobuf 解析 |
| pydantic | >=2.0.0 | 数据校验 |
| qrcode[pil] | >=7.4.2 | 扫码登录二维码生成 |
| python-socks | >=2.8.2 | SOCKS 代理支持 |
| uv | — | 比 pip 更快的依赖管理，CI 缓存友好 |
| GitHub Actions | — | CI/CD 持续集成 |

---

## 八、常见问题

### Q1: 使用 `uvx xianyu-mcp` 报 `AttributeError: 'Server' object has no attribute 'list_tools'`？

本项目未发布到 PyPI，`uvx xianyu-mcp` 会从 PyPI 安装同名第三方包。必须先 clone 仓库，使用 `uv run xianyu-mcp` 或 `python -m xianyu_mcp.server` 运行。

### Q2: `validate_login` 返回 `FAIL_SYS_USER_VALIDATE`？

当前 Cookie 已失效或不完整，或触发了更强风控校验。建议走 `qr_login_generate` / `qr_login_status` / `qr_login_cookie` 重新获取 Cookie。

### Q3: 虚拟商品无法重新上架？

虚拟商品受闲鱼 PC 端发布管控，接口会返回 `FAIL_BIZ_PC_NOT_SUPPORT_PUBLISH_OR_EDIT`。支持 PC 编辑的实物商品可正常使用 `downshelf_item` / `reshelf_item`。

### Q4: `list_my_items` 报页数超限？

把 `page_size` 调回默认值 `20`。虽然工具层做了 1~50 的参数约束，但服务端对不同账号的实际限制可能更严格，传过大可能返回 `FAIL_BIZ_FORBIDDEN`。

### Q5: 修改 Cookie 后未生效？

当前实现会在每次工具调用前重新读取 `.env`。如果修改的是仓库根目录 `.env` 或 `XIANYU_COOKIE_FILE` 指向的文件，下一次调用会自动读取新值。如客户端对 MCP 进程做了缓存，重载客户端的 MCP 服务更稳妥。

---

## 九、总结

xianyu-mcp-server 将闲鱼商品管理、会话消息、扫码登录等能力封装为 20 个标准 MCP 工具，通过风控护栏保障长期运行安全。项目基于 pyxianyu 底层库，支持 uv 和 pip 两条运行路径，可接入 Trae、Claude Desktop、Cursor、VS Code、Cherry Studio 五大客户端。适合闲鱼卖家减少重复操作、研究 MCP 落地案例、构建 AI 客服或店铺自动化工作流。

**相关链接**：

- GitHub 仓库：[DoLovya/xianyu-mcp-server](https://github.com/DoLovya/xianyu-mcp-server)
- 底层项目：[pyxianyu](https://github.com/DoLovya/xianyu-mcp-server/tree/main/third_party/pyxianyu)
- CI/CD 文档：[docs/ci-cd.md](https://github.com/DoLovya/xianyu-mcp-server/blob/main/docs/ci-cd.md)
- 鸣谢：[XianYuApis](https://github.com/cv-cat/XianYuApis)、[XianyuAutoAgent](https://github.com/shaxiu/XianyuAutoAgent)
