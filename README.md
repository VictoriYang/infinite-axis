# Paper Notes

我的论文阅读记录与总结。基于 [Hugo](https://gohugo.io/) + [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 主题，托管在 GitHub Pages。

## 首次使用

### 1. 填入你的信息（3 处）
搜索仓库里的 `EDIT_ME` / `EDIT ME`，主要在 `hugo.yaml`：
- `baseURL` → `https://<你的用户名>.github.io/paper-notes/`
- `title` / `author` / `description`
- `socialIcons` 里的 GitHub 链接

> 注意：`baseURL` 在部署时会被 GitHub Actions 自动覆盖为正确值，本地预览才会用到这里的值。

### 2. 本地预览（可选，需先装 Hugo）
```bash
# macOS 装 Hugo（extended 版，主题需要）
brew install hugo        # 若无 brew，见 https://gohugo.io/installation/

# 克隆时记得拉取主题子模块
git clone --recurse-submodules <你的仓库地址>
# 若已克隆但没主题：
git submodule update --init --recursive

hugo server -D           # 打开 http://localhost:1313
```

### 3. 推到 GitHub 并开启 Pages
```bash
git add -A
git commit -m "init paper notes site"
git remote add origin git@github.com:<你的用户名>/paper-notes.git
git push -u origin main
```
然后到仓库 **Settings → Pages → Build and deployment → Source** 选 **GitHub Actions**。
之后每次 `git push` 都会自动构建并部署。

## 写一篇新笔记
在 `content/posts/` 下新建 `.md` 文件（可复制示例笔记 `attention-is-all-you-need.md`）：

```yaml
---
title: "论文标题"
date: 2026-08-06
math: true              # 需要公式就设 true
tags: ["标签1", "标签2"]
summary: "一句话摘要，会显示在列表页。"
---
正文用 Markdown，公式用 $...$（行内）或 $$...$$（独立行）。
```

## 目录结构
```
hugo.yaml                         # 站点配置（菜单、KaTeX、搜索等）
content/
  posts/                          # 所有论文笔记放这里
  about.md  archives.md  search.md
layouts/partials/extend_head.html # KaTeX 公式注入
static/                           # 图片等静态资源
themes/PaperMod/                  # 主题（git submodule）
.github/workflows/hugo.yml        # 自动部署
```
