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
    │   └── *.jpg / *.png      # 文章图片
    └── code/                  # 子模块 → 对应项目代码仓库
```

## 写作规范

### 文章结构

每篇文章包含：

1. **元信息**：标题、标签、日期、封面图
2. **摘要**：100–200 字概述
3. **正文**：分章节，含代码和图片
4. **相关链接**：项目仓库等

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
