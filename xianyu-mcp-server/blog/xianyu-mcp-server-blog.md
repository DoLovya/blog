# 让 AI 帮我管闲鱼店铺：xianyu-mcp-server 体验分享

每天手动上架、回消息、改价格、查竞品太费时间了。这个开源项目把闲鱼能力封装成标准 MCP 工具，接到 Trae、Claude Desktop、Cursor、VS Code、Cherry Studio 这些 AI 客户端里就能直接用。

**20 个 MCP 工具 | 5 大客户端 | 扫码登录 | 风控护栏 | `uv` / `pip` 可跑**

GitHub 仓库：[DoLovya/xianyu-mcp-server](https://github.com/DoLovya/xianyu-mcp-server)

---

## 能干什么：一句话，AI 直接替你干活

接好之后，你直接说人话就行。

> “把我店里所有价格低于 50 的商品下架”

AI 会先拉商品列表，再筛选，再逐个下架。不是一股脑乱调接口，项目里已经带了风控护栏，会自动控制节奏。

> “看看最近有没有人给我发消息，有的话帮我回一下”

AI 可以先拉会话列表，再读历史消息，最后直接回复买家。对卖家来说，这已经很像一个能用的 AI 客服了。

> “帮我搜一下闲鱼上有没有二手 Switch，按最新发布排序”

AI 会直接调搜索接口，把最新发布的结果整理出来，你不用再自己打开 App 一页页翻。

我觉得这项目最省心的一点就在这儿：不是让你自己去折腾 Cookie 签名、接口参数、消息协议，而是直接把这些能力包成了 MCP 工具，AI 客户端拿来就能调。

---

## 现在到底能做什么：20 个 MCP 工具，卖家高频动作基本都包了

当前 `xianyu-mcp-server` 已开放 **20 个 MCP 工具**，大致分成 5 组。

### 1. 登录与登录态维护

| 工具 | 说明 |
|------|------|
| `validate_login` | 校验当前 Cookie 是否有效 |
| `refresh_login` | 刷新当前登录态 |
| `qr_login_generate` | 生成扫码登录二维码 |
| `qr_login_status` | 查询扫码登录状态，必要时返回验证信息 |
| `qr_login_cookie` | 在扫码成功后提取完整 Cookie |
| `qr_login_save_env` | 把扫码得到的 Cookie 直接写回 `.env` |

这套链路最大的好处就是：**第一次用不用自己抓 Cookie**。`.env` 留空启动也行，项目会引导你扫码，成功后还能直接写回环境文件。

### 2. 商品搜索、详情与店铺管理

| 工具 | 说明 |
|------|------|
| `search_items` | 按关键词搜索闲鱼商品，支持分页与排序 |
| `get_item_detail` | 获取指定商品详情 |
| `get_item_edit_detail` | 获取商品在 PC 编辑页的详情 |
| `list_my_items` | 自动翻页拉取当前账号下全部商品 |
| `downshelf_item` | 下架指定商品 |
| `reshelf_item` | 通过 PC 编辑重发布链路重新上架商品 |
| `edit_item` | 编辑商品标题、价格、描述等信息 |
| `publish_physical_item` | 发布新的实体商品，支持自动上传图片 |

如果你本来就在闲鱼卖东西，这一组其实已经很够用了：查商品、改商品、上下架、发新品，常用动作基本都在里面。

### 3. 用户信息与会话消息

| 工具 | 说明 |
|------|------|
| `get_my_profile` | 获取当前登录账号的个人资料信息 |
| `list_conversations` | 拉取最近会话列表 |
| `list_conversation_messages` | 获取指定会话历史消息 |
| `send_text_message` | 发送文本消息 |
| `send_image_message` | 发送图片消息 |

所以它就不只是“自动上下架工具”了，往客服助手、卖家助手这个方向继续做会很顺。

### 4. 媒体上传

| 工具 | 说明 |
|------|------|
| `upload_media` | 上传本地文件或 URL 素材，返回可复用媒体链接 |

这个能力既能给 `publish_physical_item` 用，也能给图片消息和后续编辑商品复用。

### 5. 风控友好的调用护栏

这部分不是单独的 MCP 工具，但很重要。仓库里已经做了请求护栏：

1. 读写分离限速：读操作和写操作使用不同节奏
2. 写操作串行化：避免多条写请求并发触发风控
3. 指数退避：遇到可疑错误自动退避重试
4. 冷却机制：识别强风控信号后主动进入冷却

自动化不等于乱调接口。我自己会更看重这种地方，因为它决定了这玩意儿到底是 Demo，还是能拿来长期跑。

---

## 为什么我觉得这个项目值得看：它不只是把接口调通了

### 价值一：标准 MCP 协议，主流 AI 客户端都能接

我比较喜欢的一点是，它走的是标准 MCP（Model Context Protocol），不是只给某一个 AI 产品单独做适配。

| 客户端 | 接入方式 |
|--------|----------|
| **Trae** | 项目内配置 `.trae/mcp.json` |
| **Claude Desktop** | 写入 `claude_desktop_config.json` |
| **Cursor** | 配置 `.cursor/mcp.json` |
| **VS Code** | 配置 `.vscode/mcp.json` |
| **Cherry Studio** | UI 中新增 MCP 服务，可选 `stdio` 或 HTTP |

说白了就是：服务跑起来之后，你想接 Trae、Cursor、Claude Desktop 还是别的客户端，都比较顺。

### 价值二：扫码登录友好，第一次使用门槛低

很多自动化项目最劝退的一步，就是上来先让你手动抓 Cookie。这个项目把扫码登录整成了完整链路：

1. 生成二维码
2. 手机扫码确认
3. 查询登录状态
4. 拿到完整 Cookie
5. 一键写回 `.env`

如果遇到人脸验证或额外风控，工具也会把状态告诉你，不会只给你一个很糊的失败提示。

### 价值三：从“会调接口”进化到“可接入真实工作流”

很多项目把接口打通就算完了，但 MCP 这种形态的好处是，它天然适合往 Agent 工作流里接。比如：

- 用 AI 批量整理店铺商品
- 用 AI 自动做竞品搜索
- 用 AI 帮你回复买家
- 后续继续扩展成消息监听 + 自动回复 Worker

这种能力一旦接进标准 MCP，后面你想继续做自动化、做编排、做助手，都会顺很多。

---

## 怎么跑起来：其实就三步

我建议直接按仓库方式来：**先 clone，再本地装依赖，再启动**。  
它和已经发到 PyPI 的 MCP 项目不太一样，现在**不能直接用 `uvx xianyu-mcp`**。

### 第一步：拉代码

```bash
git clone https://github.com/DoLovya/xianyu-mcp-server.git
cd xianyu-mcp-server
git submodule update --init --recursive
cp .env.example .env
```

`.env` 先留空也行，后面按提示走扫码登录就可以。

### 第二步：安装依赖

机器上能装 `uv` 的话，我建议优先用 `uv`：

```bash
uv pip install -e third_party/pyxianyu
uv pip install -e .
```

装不上 `uv` 也没关系，直接退回 `pip`：

```bash
pip install -e third_party/pyxianyu
pip install -e .
```

### 第三步：启动 MCP

默认走 `stdio`：

```bash
# 推荐（需要 uv）
uv run xianyu-mcp

# 或使用 pip 方案
python -m xianyu_mcp.server
```

如果要启 HTTP 模式：

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

### 一个很重要的坑

不要直接运行：

```bash
uvx xianyu-mcp
```

当前这个项目**还没有发布到 PyPI**。你如果直接跑 `uvx xianyu-mcp`，很可能会装到同名第三方包，然后开始报一堆看起来莫名其妙的问题。

---

## Trae / Cursor / Claude Desktop 怎么接

大多数 MCP 客户端都走 `stdio`。这个项目现在最稳的接法，就是让客户端直接在本地仓库里把它拉起来。

Trae 项目级配置示例：

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

如果某些客户端不支持 `${workspaceFolder}`，就改成绝对路径。比如：

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

如果你机器上确实装不上 `uv`，再退回 Python 方式：

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

这一段你记住几个点就够了：

- **Trae / Cursor / Claude Desktop / VS Code** 一般优先走 `stdio`
- **Cherry Studio** 除了 `stdio`，也可以接 HTTP 模式
- 源码开发场景优先使用 `uv --directory ${workspaceFolder} run xianyu-mcp`
- 装不上 `uv` 时，再用 `python -m xianyu_mcp.server`
- 当前不要写 `uvx xianyu-mcp`
- Cookie、环境变量这些细节，直接看 GitHub 仓库里的 README 就够了

---

## 实战里它能帮你做什么

### 场景一：批量店铺运维

> “把我店里所有价格低于 100 的商品都下架”

AI 会先调用 `list_my_items` 拉完整个商品列表，再筛选，再逐个调用 `downshelf_item`。你不用自己在 App 里来回点半天。

### 场景二：竞品搜索与价格观察

> “搜一下闲鱼上二手 Switch，按最新发布排序”

AI 直接调用 `search_items`，把最新发布结果整理出来。你后面还可以继续追问，比如：

- 哪些价格更低
- 哪些卖家在同城
- 哪些成色描述更好

### 场景三：AI 帮你做客服

> “看看最近有没有人问我商品问题，有的话帮我回一下”

这个流程会串起：

1. `list_conversations`
2. `list_conversation_messages`
3. `send_text_message` / `send_image_message`

对经常被买家重复问问题的卖家来说，这已经很接近“先让 AI 顶一轮客服”了。

### 场景四：一键发布新品

> “帮我发一个九成新 Kindle Oasis，价格 800，配这几张图”

AI 可以先上传图片，再调用 `publish_physical_item` 完成发布。对经常在电脑上整理素材的人来说，这比在手机里反复填表单顺手多了。

---

## 工程质量：至少不是随手拼的玩具项目

按现在仓库里能直接看到的信息，大致是这样：

| 指标 | 当前情况 |
|------|----------|
| MCP 工具数 | 20 |
| Python 测试矩阵 | 3.11 / 3.12 / 3.13 |
| 测试文件 | 8 |
| 单元测试 | 24 |
| 包入口 | `xianyu-mcp` |
| 依赖管理 | `uv` |
| CI | GitHub Actions 持续集成 |

更关键的是，这个项目已经不只是“几个脚本拼一起”的状态了，而是在往更标准的 Python 项目结构和 MCP 接入方式上收。对使用者来说，直接感受到的就是：

- 使用者安装成本更低
- IDE / AI 客户端接入更顺
- 同时也保留了 `uv` 和 `pip` 两条本地运行路径

---

## 目前的边界，以及后面还能怎么长

我还是倾向于先把边界说清楚，这样比较实在。

### 当前已知限制

- 虚拟商品通常受闲鱼 PC 端发布/编辑限制，当前无法稳定走 `reshelf_item` 重新上架
- 常驻消息监听与自动回复 Worker 还没有 MCP 化
- 语音、视频类消息工具还没开放出来

### 接下来最值得期待的方向

- 消息监听和自动回复拆成独立 Worker
- 更多商品类型支持
- 面向卖家的数据分析能力
- 更完善的多客户端接入文档

所以我会把它定义成：**已经能用，也很适合继续往 AI 工作流里接的闲鱼 MCP 基础设施**。

---

## 适合谁用

如果你属于下面几类人，这个项目大概率会挺对胃口：

- 经常在闲鱼卖二手、想减少重复操作的人
- 喜欢把日常工作流交给 AI 自动化的人
- 正在研究 MCP 落地案例的人
- 想做 AI 客服、AI 店铺助手、Agent 自动运维的人

它不是那种演示一下就结束的 Demo，而是真的有机会继续长进真实卖家工作流里的。

---

## 最后一句

如果你最近正好在折腾 AI 自动化、MCP，或者就是想少做点重复的闲鱼操作，我觉得这个项目挺值得自己跑一遍。

先按 README 当前推荐方式跑起来：

```bash
git clone https://github.com/DoLovya/xianyu-mcp-server.git
cd xianyu-mcp-server
git submodule update --init --recursive
cp .env.example .env
uv pip install -e third_party/pyxianyu
uv pip install -e .
uv run xianyu-mcp
```

如果你机器上没有 `uv`，就把最后三步换成：

```bash
pip install -e third_party/pyxianyu
pip install -e .
python -m xianyu_mcp.server
```

GitHub 仓库：[DoLovya/xianyu-mcp-server](https://github.com/DoLovya/xianyu-mcp-server)

它吸引我的点也不只是“能连上接口”，而是登录、商品、消息、上传、风控这些关键拼图，已经被它接得比较完整了。
