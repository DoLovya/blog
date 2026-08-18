# 多平台博客发布指南

## 方案对比

| 方案 | 复杂度 | 维护成本 | 适用场景 |
|------|--------|---------|---------|
| **图床 + Markdown** | ⭐⭐ | 低 | 个人博客，多平台发布 |
| **静态博客站** | ⭐⭐⭐⭐ | 中 | 有域名，长期运营 |
| **第三方工具** | ⭐⭐⭐ | 中 | 批量发布需求 |

---

## 方案一：GitHub + jsDelivr 图床（推荐）

### 1. 图片放在仓库内

```bash
# 图片直接放到 {项目}/blog/ 目录下
# door-clicker/blog/architecture-diagram.jpg
# door-clicker/blog/web-ui-screenshot.jpg
```

### 2. 获取 CDN 链接

```
https://cdn.jsdelivr.net/gh/DoLovya/blog@main/{path}
```

**示例**：
```
https://cdn.jsdelivr.net/gh/DoLovya/blog@main/door-clicker/blog/architecture-diagram.jpg
```

> **刷新缓存**：push 后图片未立即更新时，访问 `https://purge.jsdelivr.net/gh/DoLovya/blog@main/{path}` 手动刷新。

### 3. 替换文档图片路径

```markdown
# 将本地路径（不推荐用于发布平台）
![架构图](./architecture-diagram.jpg)

# 替换为 CDN 路径（所有博客平台通用）
![架构图](https://cdn.jsdelivr.net/gh/DoLovya/blog@main/door-clicker/blog/architecture-diagram.jpg)
```

### 4. 发布到各平台

#### CSDN
1. 新建文章，选择 Markdown 编辑器
2. 粘贴内容，图片自动加载
3. 设置标签：ESP8266、MQTT、物联网、门禁系统
4. 发布

#### 知乎
1. 新建文章，粘贴 Markdown 内容
2. 图片自动显示
3. 添加话题：#ESP8266 #MQTT #物联网
4. 发布

#### 稀土掘金
1. 新建文章，Markdown 模式
2. 粘贴内容
3. 选择技术领域
4. 发布

#### 博客园
1. 随笔 → 新建 → Markdown
2. 粘贴内容
3. 添加标签
4. 发布

---

## 方案二：静态博客站

### 推荐工具

| 工具 | 特点 | 适用 |
|------|------|------|
| Hugo | 速度快，模板丰富 | 个人站点 |
| Jekyll | GitHub Pages 原生支持 | 技术博客 |
| Hexo | 插件多，Node.js | 国内用户 |
| VitePress | Vue 驱动，现代化 | 技术文档 |

### 快速开始（Hugo）

```bash
# 安装 Hugo
brew install hugo

# 创建站点
hugo new site my-blog
cd my-blog

# 添加主题
git init
git submodule add https://github.com/bulletmark/hugo-theme-minima themes/minima
echo 'theme = "minima"' >> hugo.toml

# 创建文章
hugo new posts/door-clicker.md

# 启动预览
hugo server

# 构建发布
hugo --minify
```

### 部署到 GitHub Pages

```yaml
# .github/workflows/pages.yml
name: Deploy Blog

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: hugo --minify
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

---

## 方案三：第三方工具

### 推荐工具

| 工具 | 平台支持 | 说明 |
|------|---------|------|
| [Markdown Nice](https://www.mdnice.com/) | 微信公众号、知乎、CSDN 等 | 在线编辑器，支持多平台导出 |
| [iWiki](https://iwiki.qq.com/) | 内部使用 | 腾讯出品，支持 Markdown |
| [语雀](https://yuque.com/) | 多平台导出 | 支持导出到主流平台 |
| [Notion](https://www.notion.so/) | 导出功能 | 支持导出 Markdown，需手动发布 |

### Markdown Nice 使用

1. 访问 https://www.mdnice.com/
2. 粘贴 Markdown 内容
3. 选择主题样式
4. 一键复制到目标平台

---

## 图片处理最佳实践

### 1. 图片命名规范

```
{topic}-{description}.{ext}

示例：
- circuit-diagram.jpg      # 接线图
- architecture-diagram.jpg # 架构图
- web-ui-screenshot.jpg    # 网页截图
- flow-chart.png           # 流程图
```

### 2. 图片尺寸建议

| 用途 | 推荐尺寸 | 格式 |
|------|---------|------|
| 架构图 | 1920x1080 | PNG/JPG |
| 接线图 | 1280x960 | PNG/JPG |
| 截图 | 实际尺寸 | PNG |
| 流程图 | 1080px 宽 | PNG/SVG |

### 3. 图床选择

| 图床 | 优点 | 缺点 |
|------|------|------|
| GitHub + jsDelivr | 免费，稳定 | 国内访问可能慢 |
| 七牛云 | 国内速度快 | 需付费 |
| 阿里云 OSS | 稳定可靠 | 需付费 |
| SM.MS | 免费简单 | 有流量限制 |
| Imgur | 免费 | 国内访问慢 |

---

## 发布检查清单

- [ ] 图片已上传到图床
- [ ] 文档中图片路径已替换为 CDN 链接
- [ ] 代码块格式正确
- [ ] 表格显示正常
- [ ] 链接可正常访问
- [ ] 标题层级正确
- [ ] 标签/话题已添加
- [ ] 摘要已编写

---

## 注意事项

1. **各平台差异**
   - CSDN 支持大部分 Markdown 语法
   - 知乎对某些 HTML 标签有限制
   - 稀土掘金不支持 Mermaid 图表
   - 微信公众号格式最严格

2. **图片加载**
   - 确保 CDN 在目标平台可访问
   - 国内平台建议使用国内图床
   - 可考虑使用多 CDN 备份

3. **版权声明**
   - 图片注明来源
   - 代码使用 MIT/Apache 等开源协议
   - 如需转载请注明出处

---

## 总结

**推荐方案**：GitHub + jsDelivr 图床 + Markdown Nice 编辑器

**发布流程**：
1. 图片上传到 GitHub 图床
2. 使用 Markdown Nice 编辑和美化文档
3. 一键复制到各平台
4. 检查格式后发布

这样可以实现"一次写作，多平台发布"，大幅提升效率！
