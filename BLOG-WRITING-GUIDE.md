# 博客写作规范

个人技术博客仓库，聚合多个项目的技术文章。

## 仓库结构

```
blog/
├── README.md                  # 仓库说明
├── BLOG-WRITING-GUIDE.md      # 本文档：写作规范
├── PUBLISH-GUIDE.md           # 多平台发布指南
├── AI-CONTENT-SPEC.md         # AI 生成内容规范
├── .gitmodules                # 子模块配置
│
└── {项目名}/                  # 每个项目一个目录
    ├── blog/                  # 博客内容（本仓库管理）
    │   ├── {项目名}-blog.md   # 博客主文章
    │   └── images/            # 本文章所有图片（*.jpg / *.png）
    ├── code/                  # 子模块 → 对应项目代码仓库
    └── PUBLISH.md             # （推荐）项目专属发布清单
```

---

## 推荐的仓库目录组织方式

### 1. 两种仓库层级（按项目数量选择）

#### 方案 A · 单层聚合（当前在用，项目数 ≤ 6 推荐）

所有项目平级放在 `blog/` 根目录，扁平直观：

```
blog/
├── README.md
├── BLOG-WRITING-GUIDE.md
├── PUBLISH-GUIDE.md
├── AI-CONTENT-SPEC.md
├── .gitmodules
│
├── door-clicker/
│   ├── blog/door-clicker-blog.md
│   ├── blog/images/*.jpg
│   ├── code/           (submodule)
│   └── PUBLISH.md
│
├── esp-weather-station/
├── my-blog-site/
└── ...
```

#### 方案 B · 按领域分组（项目数 ≥ 7，或跨多个垂直领域推荐）

当项目覆盖 IoT / Web / AI 等多个方向时，加一层领域目录，查找和维护更清爽：

```
blog/
├── iot/                     # 物联网 / 硬件类
│   ├── door-clicker/
│   │   ├── blog/, code/, PUBLISH.md
│   └── esp-weather-station/
├── web/                     # Web / 前后端项目
│   ├── my-blog-site/
│   └── dashboard-pro/
└── ai/                      # AI / 工具 / Agent 项目
    ├── local-llm-assistant/
    └── trae-skill-collection/
```

**迁移建议**：新项目先用方案 A；当项目达到 6+ 后再一次性迁移到方案 B。迁移时用 `git mv` 保留历史，同步更新文章中所有 jsDelivr CDN 图片路径（结构变化后 CDN 路径也会变化，需要重新 push 并刷新缓存）。

### 2. 单个项目目录模板（新建任何项目时按此复制）

```
{项目名-kebab-case}/
├── blog/                        # 博客正文（blog 仓库管理，独立于代码）
│   ├── {slug}-blog.md           # 正文主文章（文件名 = 项目 + -blog.md）
│   ├── .gitignore               # .DS_Store / Thumbs.db / ~$临时文件
│   └── images/                  # ⚠️ 图片必须统一放这里（禁止平铺在 blog/ 根）
│       ├── architecture-diagram.jpg
│       ├── hardware-installation.jpg
│       ├── web-ui-mobile.jpg
│       ├── firmware-config.jpg
│       └── web-config.jpg
├── code/                        # submodule → git@github.com:{user}/{repo}.git
└── PUBLISH.md                   # 项目专属发布清单（标题推荐/标签/摘要/CDN直链/检查清单）
```

**创建新项目的 Shell 模板**（复制执行即可）：

```bash
# 1. 建目录结构
PROJECT=my-new-project          # TODO: 改成你自己的项目名（kebab-case）
mkdir -p "$PROJECT/blog/images"
touch "$PROJECT/blog/images/.gitkeep"
cp door-clicker/PUBLISH.md "$PROJECT/PUBLISH.md"   # 从现有项目复制发布清单模板
cp door-clicker/blog/.gitignore "$PROJECT/blog/.gitignore"

# 2. 添加代码 submodule（替换成实际的仓库地址）
git submodule add git@github.com:DoLovya/${PROJECT}.git "$PROJECT/code"

# 3. 新建空文章
touch "$PROJECT/blog/${PROJECT}-blog.md"
```

### 3. 命名规则（全部强制）

| 项 | 规则 | 正确示例 | 错误示例 |
|----|------|---------|---------|
| 项目目录名 | **kebab-case**，纯 ASCII，无空格/中文/下划线 | `door-clicker` | `DoorClicker`、`智能门禁`、`door_clicker` |
| 正文 md 文件名 | `{项目slug}-blog.md` | `door-clicker-blog.md` | `blog.md`、`Door Clicker Blog.md` |
| 图片文件名 | kebab-case + 内容描述，禁止中文 | `hardware-installation.jpg` | `接线图.jpg`、`截图1.png` |
| 领域目录名（方案 B） | 全小写短单词 | `iot`、`web`、`ai`、`mobile` | `IoT_Projects`、`物联网项目` |

### 4. 图片存放硬性规定

- **禁止** `{项目名}/blog/*.jpg` 平铺，一律放 `{项目名}/blog/images/` 下
- **禁止**同一篇文章的图片分散在多个不同路径，`images/` 是**唯一**出口
- **禁止**把 AI 草稿图、未裁剪的原 PNG、带个人水印的版本直接 commit，仅保留最终发布用 JPG

> 为什么：这样可以保证所有 CDN 路径统一为 `.../blog/images/<filename>`，后续替换图床 / 迁移目录时不需要手动改几十条引用。

---

## 写作规范

### 文章结构（每篇必写 13 章，与 AI-CONTENT-SPEC.md 对齐）

每篇文章按此顺序组织，**不得跳过「项目背景 / 动机」章节**：

1. **项目背景 / 动机（为什么做）** ← 本节必读，见下方写作模板
2. **项目简介 / 核心特性**
3. **系统架构**（含架构图）
4. **硬件准备**（硬件类）/ **技术选型**（软件类）
5. **快速开始**（环境搭建 + Hello World 跑通）
6. **功能详解**（逐模块拆解）
7. **API 接口**（有对外 API 时必写）
8. **项目结构**（目录树 + 关键模块说明）
9. **部署上线**（生产部署步骤 / CI/CD）
10. **技术栈**（版本表）
11. **常见问题**（FAQ，至少 3 条）
12. **总结**
13. **相关链接**（项目源码等）

### 项目背景 / 动机 — 4 段写作模板

> ⚠️ **必须真实，不得编造。** 字数 200~350 字为宜，建议附至少 1 张实物图或场景图。

| 段落 | 写什么 | 反例（禁止写） |
|------|--------|---------------|
| 1. 真实生活 / 工作场景 | 描述你**现在实际遇到的场景**：比如"住老式单元楼，每次朋友来要下楼按对讲机" | "为了学习 ESP8266 我想做一个项目" |
| 2. ≥2 个具体痛点 | 每个痛点 ≥1 句话，讲清"麻烦在哪里"：如"手上拎东西时掏钥匙特别狼狈" | "现有方案不好用"（空泛） |
| 3. 核心思路 | 用什么技术栈 / 组件 **解决了什么问题**：如"SG90 舵机贴在门铃按键旁代替手按 + MQTT 远程控制" | 只列技术名词、不解释解决什么 |
| 4. 带来的好处 | 做完后**改善了什么**：如"不用带钥匙、不用跑到门口" | "具有革命性意义"（夸大） |

**禁止出现的字样**：颠覆、革命性、最强、宇宙第一、完美、秒杀、吊打一切等夸张词一律禁用。

### 图片规范

- 命名：`{topic}-{description}.jpg`
- 尺寸：宽度不超过 1920px
- 格式：JPG（照片）/ PNG（图表）
- 引用：使用 jsDelivr CDN 链接
  ```
  https://cdn.jsdelivr.net/gh/DoLovya/blog@main/{path}
  ```

### 标签体系

| 标签 | 用途 |
|------|------|
| ESP8266 | 单片机相关 |
| MQTT | 物联网通信 |
| 实战 | 项目实战 |
| 开源 | 开源项目 |
| 物联网 | IoT 相关 |

## 发布流程

1. 图片放到项目下的 `blog/` 目录
2. 图片路径替换为 jsDelivr CDN 链接
3. 使用 Markdown Nice 编辑美化
4. 发布到各平台（CSDN、知乎、掘金）

## 相关链接

- GitHub: https://github.com/DoLovya
- Blog 仓库: https://github.com/DoLovya/blog
