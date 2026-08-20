# pyxianyu：闲鱼 HTTP / WebSocket 能力库

> 封装闲鱼 Web 端签名、登录态、商品、消息等接口，为上层应用提供统一调用抽象
>
> 📦 **项目源码**：[github.com/DoLovya/pyxianyu](https://github.com/DoLovya/pyxianyu)

## 〇、项目背景（为什么做这个）

做闲鱼自动化时，业务代码还没写几行，先卡在了底层接口上：

1. **请求签名不能裸调**：闲鱼 Web 端所有 HTTP 接口的参数里都带一个 `sign` 字段，依赖 Cookie 中的 `_m_h5_tk` token 做_MD5 签名。直接调接口不传 sign 或 token 过期，服务端直接拒绝。
2. **WebSocket 消息协议复杂**：闲鱼私信走 WebSocket，消息体用 Protobuf 编码 + base64 + sign 签名，不是标准的 JSON 协议。想做 AI 客服或自动回复，光协议层就要啃很久。
3. **Cookie / token 会过期**：常驻进程跑一段时间后 token 失效，没有自动刷新机制就会突然全部接口报错。

既然这些是所有闲鱼自动化项目的前置问题，为什么不封成一个库？

于是用 **Python + requests + websockets** 的方案：`XianyuClient` 统一处理签名生成和 Session/Cookie 管理，`XianyuApis` 聚合商品/搜索/鉴权/媒体等 HTTP API，`XianyuLive` 处理 WebSocket 消息收发。底层库封稳后，上层项目（如 xianyu-mcp-server）只管组装业务，不用再重复处理签名和协议细节。

## 一、项目简介

pyxianyu 是一个闲鱼底层 HTTP / WebSocket 能力库，封装了闲鱼 Web 端的签名生成、登录态管理、商品操作、搜索、媒体上传、私信收发等能力。通过 `pip install pyxianyu` 即可安装使用，也可作为 git submodule 被上层项目集成。

### 核心特性

- 🔑 **签名自动生成**：`XianyuClient` 统一处理 `sign` 签名（MD5），业务层只需组装 `data`
- 🔄 **登录态管理**：Cookie 注入 + token 自动刷新，常驻进程不掉线
- 📦 **模块化 API**：鉴权、商品、搜索、用户、媒体分模块，`XianyuApis` 统一聚合
- 💬 **WebSocket 消息**：逆向还原闲鱼私信协议（sign + base64 + Protobuf），支持文本/图片/音频消息
- 🐍 **广兼容**：CPython 3.9~3.13 + PyPy 3.10 双支持，CI 持续验证
- 📦 **PyPI 已发布**：`pip install pyxianyu` 即可使用

---

## 二、系统架构

<img src="https://cdn.jsdelivr.net/gh/DoLovya/blog@main/pyxianyu/blog/images/architecture-diagram.jpg" alt="系统架构图" width="70%">

### 分层说明

| 层 | 职责 | 关键类 |
|----|------|--------|
| **应用层** | MCP 服务、机器人、运营脚本 | xianyu-mcp-server 等 |
| **聚合层** | 对外统一入口 | `XianyuApis`（HTTP）、`XianyuLive`（WebSocket） |
| **模块层** | 按业务域拆分 | `AuthApi`、`ItemApi`、`SearchApi`、`UserApi`、`MediaApi` |
| **核心层** | 签名、Session、Cookie 管理 | `XianyuClient` |
| **协议层** | HTTP（mtop）+ WebSocket（Protobuf） | requests / websockets |

### 数据流

1. **应用层 → XianyuApis**：调用 `get_item_info(item_id)` 等方法
2. **XianyuApis → 模块层**：委托给 `ItemApi(client)` 等子模块
3. **模块层 → XianyuClient**：组装 mtop 参数 → `build_signed_form` 生成 sign → 发送 HTTP 请求
4. **XianyuClient → 闲鱼服务端**：通过 `requests.Session`（携带 Cookie）发送签名后的请求

WebSocket 消息走独立链路：`XianyuLive` 建立 WebSocket 连接 → `msgpack` + Protobuf 编解码 → 收发私信。

---

## 三、技术选型

- **requests 而非 httpx**：闲鱼 HTTP 接口都是同步请求，`requests.Session` 天然支持 Cookie 持久化，简单直接。
- **websockets 而非 aiohttp**：闲鱼私信走 WebSocket，`websockets` 库 API 简洁，兼容 CPython 和 PyPy。
- **msgpack + blackboxprotobuf**：闲鱼 WebSocket 消息体用 Protobuf 编码 + msgpack 序列化，必须用对应库解析。
- **hatchling 构建**：比 setuptools 更现代，wheel 包路径直接指向 `src/pyxianyu`，src 布局干净。

---

## 四、快速开始

### 4.1 安装

```bash
# pip
pip install pyxianyu

# uv
uv pip install pyxianyu

# uvx（一次性验证）
uvx --from pyxianyu python -c "import pyxianyu; print(pyxianyu.__version__)"
```

### 4.2 配置 Cookie

Cookie 通过环境变量注入：

```env
# .env
XIANYU_COOKIE=完整_cookie_字符串
```

> 💡 Cookie 必须是登录 [goofish.com](https://www.goofish.com) 后的状态，否则无法获取消息。

### 4.3 最小示例

```python
import os

from pyxianyu.xianyu_apis import XianyuApis
from pyxianyu.utils.xianyu_utils import generate_device_id, trans_cookies


def main():
    cookie_str = os.environ["XIANYU_COOKIE"]
    cookies = trans_cookies(cookie_str)
    user_id = cookies.get("unb", "0")
    device_id = generate_device_id(user_id)
    api = XianyuApis(cookies, device_id)

    token_result = api.get_token()
    print("token ok:", bool(token_result.get("data")))

    nav_result = api.get_user_page_nav()
    print(nav_result)


if __name__ == "__main__":
    main()
```

运行：

```bash
python demo.py
```

### 4.4 Docker 部署

```bash
# 构建镜像
docker build -t pyxianyu .

# 以环境变量方式运行
docker run -it pyxianyu
```

默认入口为 `python -m pyxianyu.xianyu_live`，启动后进入消息监听模式。

---

## 五、功能详解

pyxianyu 对外暴露两个聚合入口：`XianyuApis`（HTTP 能力）和 `XianyuLive`（WebSocket 消息）。

### 5.1 鉴权

| 方法 | 参数 | 说明 |
|------|------|------|
| `get_token()` | 无 | 校验登录态并换取 `accessToken` |
| `refresh_token()` | 无 | 刷新当前登录态 |

### 5.2 商品管理

| 方法 | 参数 | 说明 |
|------|------|------|
| `get_item_info(item_id)` | `item_id: str` | 获取商品详情 |
| `get_user_items(user_id, ...)` | `user_id, page_number=1, page_size=20, ...` | 获取指定用户某一页商品 |
| `get_all_user_items(user_id, page_size=20)` | `user_id, page_size=20` | 自动翻页聚合全部商品 |
| `downshelf_item(item_id)` | `item_id: str` | 下架指定商品 |
| `prepublish_check(item_id=None)` | `item_id: str = None` | 发布前校验 |
| `preget(item_id=None, source_id=None, ...)` | `item_id, source_id, publish_scene, bizcode` | 获取发布/编辑预置参数 |
| `get_item_edit_detail(item_id)` | `item_id: str` | 获取商品 PC 编辑页详情 |
| `edit_item(payload)` | `payload: dict` | 调用 PC 编辑接口提交商品 |
| `publish_item(payload)` | `payload: dict` | 发布全新商品（PC 端发布链路） |
| `build_reshelf_payload(edit_detail_result, ...)` | `edit_detail_result, item_id, source_id` | 基于编辑详情构造重发布 payload |
| `reshelf_item(item_id, source_id=None)` | `item_id, source_id` | 一步完成"读取编辑详情 → 重发布" |

### 5.3 搜索

| 方法 | 参数 | 说明 |
|------|------|------|
| `search_items(keyword, ...)` | `keyword, page_number=1, rows_per_page=20, sort_field="", sort_value=""` | 按关键词搜索闲鱼商品 |

### 5.4 用户

| 方法 | 参数 | 说明 |
|------|------|------|
| `get_user_page_nav()` | 无 | 获取当前登录用户信息/个人页导航 |

### 5.5 媒体上传

| 方法 | 参数 | 说明 |
|------|------|------|
| `upload_media(media_path)` | `media_path: str` | 上传图片/视频/音频素材 |

### 5.6 WebSocket 消息（XianyuLive）

`XianyuLive` 负责私信实时收发，内部通过 WebSocket 连接闲鱼消息服务端：

| 方法 | 说明 |
|------|------|
| `init(ws)` | WebSocket 初始化注册 |
| `heart_beat(ws)` | 心跳保活 |
| `user_alive()` | HTTP 登录态保活 |
| `create_chat(ws, toid, item_id)` | 创建单聊会话 |
| `send_msg(ws, cid, toid, message)` | 发送文本/图片消息 |
| `send_msg_once(toid, item_id, send_message)` | 单次发送消息（自动建会话 + 发送 + 关闭） |
| `list_all_conversations(cid)` | 拉取指定会话历史消息 |
| `main()` | 启动消息监听主循环 |
| `handle_message(message, websocket)` | 处理收到的消息 |

消息类型构造器：

| 构造器 | 说明 |
|--------|------|
| `make_text(text)` | 构造文本消息 |
| `make_image(url, width=0, height=0)` | 构造图片消息 |
| `make_audio(url, duration_ms=0)` | 构造音频消息 |

> 💡 `XianyuLive` 需由上层服务调度，不宜直接嵌入阻塞主循环。

---

## 六、部署上线

### 6.1 环境要求

- Python 3.9+
- 闲鱼登录后的完整 Cookie（必须包含 `_m_h5_tk` 字段）

### 6.2 CI/CD

项目使用 GitHub Actions 持续集成，配置文件为 `.github/workflows/ci.yml`：

| 工作流 | 触发条件 | 说明 |
|--------|----------|------|
| `ci.yml` → cpython | push、PR | CPython 3.9~3.13 矩阵：安装 + 导入 + 编译 + 单测 |
| `ci.yml` → pypy-smoke | push、PR | PyPy 3.10：构建 wheel + 安装 + 导入 + 编译 + 单测 |
| `release.yml` | tag `vX.Y.Z` | Trusted Publishing 自动发布到 PyPI |

**CI 关键步骤**：

1. Checkout 代码
2. 安装依赖 `pip install -r requirements.txt && pip install -e .`
3. Smoke 导入验证（`import pyxianyu` 及子模块）
4. `python -m compileall -q src` 编译检查
5. `python -m unittest discover -s tests -v` 单元测试

### 6.3 PyPI 发布

项目已发布到 PyPI，使用 Trusted Publishing（无需手动管理 API Token）：

1. 在 PyPI 创建项目 `pyxianyu`
2. 在 PyPI 的 Publishing 设置中新增 Trusted Publisher（指向 `DoLovya/pyxianyu` 仓库 + `release.yml`）
3. 推送 tag `vX.Y.Z` 触发 Release workflow 自动构建并发布

### 6.4 作为 submodule 集成

上层项目（如 xianyu-mcp-server）可以通过 git submodule 集成 pyxianyu 用于开发调试：

```bash
git submodule add https://github.com/DoLovya/pyxianyu.git third_party/pyxianyu
pip install -e third_party/pyxianyu
```

---

## 七、技术栈与选型

| 技术 | 版本 | 选型理由 |
|------|------|----------|
| Python | >=3.9 | 广泛兼容，CPython + PyPy 双支持 |
| requests | >=2.31.0 | 同步 HTTP 请求，Session 原生支持 Cookie 持久化 |
| loguru | >=0.7.2 | 轻量日志，支持文件轮转和异步写入 |
| websockets | >=12.0 | WebSocket 通信，API 简洁，兼容 PyPy |
| msgpack | >=1.0.0 | 消息序列化，闲鱼 WebSocket 协议使用 |
| blackboxprotobuf | >=1.0.1 | Protobuf 解析，闲鱼消息体使用 Protobuf 编码 |
| typing-extensions | >=4.0.0 | 类型标注兼容旧版 Python |
| hatchling | — | 构建后端，src 布局，生成 wheel/sdist |
| GitHub Actions | — | CI/CD + Trusted Publishing |

---

## 八、常见问题

### Q1: 接口报签名失败？

`sign` 依赖 Cookie 中的 `_m_h5_tk` 字段。如果 Cookie 缺少这个字段，签名会直接失败。确保 Cookie 是登录 [goofish.com](https://www.goofish.com) 后的完整状态。

### Q2: `get_token()` 一直重试？

`get_token()` 内部有重试逻辑，但需要限制次数。如果 token 过期且 refresh 也失败，不应进入死循环。检查 Cookie 是否已完全失效，必要时重新获取。

### Q3: 编辑商品提交后布尔值报错？

闲鱼 PC 编辑接口中，部分布尔字段实际是字符串（如 `"true"` / `"false"`）。提交前需要做布尔值归一化，否则服务端会拒绝。

### Q4: 虚拟商品无法重新上架？

`reshelf_item`、`edit_item`、`publish_item` 均走 PC 端发布/编辑链路。闲鱼对虚拟商品的 PC 端发布有管控，可能返回 `FAIL_BIZ_PC_NOT_SUPPORT_PUBLISH_OR_EDIT`。实物商品可正常使用。

### Q5: Cookie 写在代码里还是环境变量？

必须走环境变量 `XIANYU_COOKIE`，不要写死在代码里。上层项目可通过 `.env` 文件管理，pyxianyu 每次调用前会重新读取。

---

## 九、总结

pyxianyu 先解决的是闲鱼自动化的前置问题——签名、请求、鉴权和消息协议，而不是先堆接口数量。模块拆开后对外聚合成 `XianyuApis`（HTTP）和 `XianyuLive`（WebSocket），业务代码会干净很多。底层库封稳后，后面接 MCP、机器人、运营脚本才省事。

**相关链接**：

- GitHub 仓库：[DoLovya/pyxianyu](https://github.com/DoLovya/pyxianyu)
- PyPI 包：[pypi.org/project/pyxianyu](https://pypi.org/project/pyxianyu/)
- 上层项目：[DoLovya/xianyu-mcp-server](https://github.com/DoLovya/xianyu-mcp-server)
- 鸣谢：[XianYuApis](https://github.com/cv-cat/XianYuApis)
