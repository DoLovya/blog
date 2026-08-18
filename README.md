# DoLovya's Tech Blog

个人技术博客仓库，聚合多个项目的技术文章。

## 目录说明

每个项目包含 `blog/`（博客内容）和 `code/`（项目代码子模块）两部分。

```
blog/
├── README.md               # 本文件，仓库说明
├── .gitmodules             # 子模块配置
├── .gitignore
├── door-clicker/           # 智能门禁系统系列
│   ├── blog/               # 博客内容（本仓库管理）
│   └── code/               # 子模块 → door-clicker 原始仓库
└── ...                     # 其他项目系列
```

## 常用命令

```bash
# 克隆含子模块的完整仓库
git clone --recurse-submodules <url>

# 更新所有子模块到最新
git submodule update --remote --merge

# 更新指定子模块
git submodule update --remote door-clicker/code

# 添加新项目（blog + code 子模块）
mkdir -p <project>/blog
git submodule add <repo-url> <project>/code
```

## 写作规范

### 文章结构

每篇文章包含：

1. **元信息**：标题、标签、日期、封面图
2. **摘要**：100-200 字概述
3. **正文**：分章节，含代码和图片
4. **相关链接**：项目仓库、在线演示等

### 图片规范

- 命名：`{topic}-{description}.jpg`
- 尺寸：宽度不超过 1920px
- 格式：JPG（照片）/ PNG（图表）

### 标签体系

| 标签 | 用途 |
|------|------|
| ESP8266 | 单片机相关 |
| MQTT | 物联网通信 |
| 实战 | 项目实战 |
| 开源 | 开源项目 |
| 物联网 | IoT 相关 |

## 发布流程

1. 图片上传到对应项目的 `blog/` 目录
2. 使用 GitHub 图床：`https://raw.githubusercontent.com/<user>/<repo>/main/{path}`
3. 替换文档中的图片路径
4. 使用 Markdown Nice 编辑美化
5. 发布到各平台（CSDN、知乎、掘金）

## 相关链接

- GitHub: https://github.com/DoLovya
- Door Clicker: https://github.com/DoLovya/door-clicker
