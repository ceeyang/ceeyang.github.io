# ceeyang.github.io

> 🌐 个人博客源码 — [ceeyang.com](https://ceeyang.com)

**你的 AI 产品合伙人** · 全栈开发 · AI 开发 · 一人公司实践中 · 数字游民

## 技术栈

| 工具 | 说明 |
|------|------|
| [Hugo](https://gohugo.io/) | 静态站点生成器（v0.160.1） |
| [PaperMod](https://github.com/adityatelange/hugo-PaperMod) | Hugo 主题 |
| GitHub Actions | 自动构建 & 部署 |
| GitHub Pages | 站点托管 |

## 本地开发

**前置条件：** 安装 [Hugo Extended](https://gohugo.io/installation/)

```bash
# 克隆仓库（含子模块）
git clone --recurse-submodules https://github.com/ceeyang/ceeyang.github.io.git
cd ceeyang.github.io

# 启动本地开发服务器（development 环境，链接指向 localhost）
hugo serve
# 或：hugo server -D
```

访问 [http://localhost:1313](http://localhost:1313) 预览站点。

## 目录结构

```
.
├── content/          # 博客文章与页面（Markdown）
│   ├── posts/        # 文章
│   └── about/        # 关于页面
├── assets/           # 样式、脚本等资源
├── static/           # 静态文件（图片等）
├── layouts/          # 自定义布局模板
├── themes/           # Hugo 主题（Git Submodule）
│   └── PaperMod/
└── hugo.toml         # 站点配置
```

## 写作

```bash
# 新建一篇文章
hugo new posts/my-post.md
```

文章 Markdown 文件位于 `content/posts/`，使用 Front Matter 配置标题、日期、标签等元数据。

## 部署

推送到 `master` 分支后，GitHub Actions 会自动：

1. 使用 Hugo 构建静态站点
2. 将产物发布到 `gh-pages` 分支
3. 通过自定义域名 [ceeyang.com](https://ceeyang.com) 提供访问

## 联系

- GitHub: [@ceeyang](https://github.com/ceeyang)
- Twitter: [@ceeyang_crypto](https://twitter.com/ceeyang_crypto)
