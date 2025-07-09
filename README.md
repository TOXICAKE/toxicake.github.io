# N0P3's Blog

这是使用 Hexo + GitHub Pages 搭建的个人技术博客。

## 博客介绍

专注于网络安全、CTF竞赛、技术研究的个人博客，包含：

- 🔐 网络安全技术分享
- 🏆 CTF 比赛解题记录  
- 💻 编程技术心得
- 🛠️ 工具开发经验

## 快速开始

### 本地运行

```bash
# 安装依赖
npm install

# 启动本地服务器
npm run server

# 访问 http://localhost:4000/N0P3-sBlog/
```

### 新建文章

```bash
# 创建新文章
hexo new "文章标题"

# 创建新页面
hexo new page "页面名称"
```

### 部署到 GitHub Pages

```bash
# 生成静态文件
npm run build

# 部署到 GitHub Pages (需要先配置 GitHub 仓库)
npm run deploy
```

## 部署说明

### 1. 创建 GitHub 仓库

✅ 您已经有仓库：`toxicake.github.io`

### 2. 配置 GitHub Pages

1. 进入仓库的 Settings > Pages
2. Source 选择 "Deploy from a branch"
3. Branch 选择 `gh-pages`
4. 点击 Save

### 3. 配置已完成

配置已经更新为您的仓库：

```yaml
url: https://toxicake.github.io
deploy:
  type: git
  repo: https://github.com/TOXICAKE/toxicake.github.io.git
  branch: gh-pages
```

### 4. 自动部署

推送代码到 `main` 分支后，GitHub Actions 会自动构建并部署博客到 GitHub Pages。

## 技术栈

- **静态站点生成器**: [Hexo](https://hexo.io/)
- **主题**: Landscape (默认主题)
- **部署平台**: GitHub Pages
- **自动化部署**: GitHub Actions

## 目录结构

```
├── .github/workflows/    # GitHub Actions 配置
├── source/              # 源文件目录
│   ├── _posts/         # 博客文章
│   └── _drafts/        # 草稿文章
├── themes/             # 主题目录
├── public/             # 生成的静态文件
├── scaffolds/          # 文章模板
├── raw/               # 原始 markdown 文件
└── _config.yml        # Hexo 配置文件
```

## 贡献

如果你发现任何问题或有改进建议，欢迎提交 Issue 或 Pull Request。

## 许可证

本项目采用 MIT 许可证。

---

📧 联系方式：[your-email@example.com]  
🔗 博客地址：[https://toxicake.github.io]
