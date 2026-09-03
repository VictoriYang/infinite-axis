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

## 评论
文章页可以挂 [giscus](https://giscus.app)（评论存在本仓库的 GitHub Discussions 里，读者用 GitHub 账号登录即可留言）。
`hugo.yaml` 里 `params.giscus.repo` 为空时评论区不渲染，所以默认是关的。开启步骤：

1. 仓库 **Settings → General → Features** 勾选 **Discussions**。
2. 安装 [giscus app](https://github.com/apps/giscus) 并授权给本仓库。
3. 打开 <https://giscus.app>，填入 `VictoriYang/infinite-axis`，Discussion 分类选 **Announcements**，
   页面下方会生成一段配置，从中抄出 `data-repo-id` 和 `data-category-id`。
4. 把四个值填回 `hugo.yaml`：

```yaml
giscus:
  repo: "VictoriYang/infinite-axis"
  repoId: "R_kgDO..."
  category: "Announcements"
  categoryId: "DIC_kwDO..."
```

映射方式是 `pathname`，即每篇文章按 URL 路径对应一条 Discussion；主题跟随站点亮/暗自动切换。

## 侧栏
宽屏（≥1240px）时正文两侧各挂一栏，窄屏自动折到正文下方，内容不丢。

- 文章页：左栏是目录 + 阅读进度条，右栏依次是参考文献、系列、相关笔记。
  参考文献由 JS 从正文里抽外链去重生成，不用手写；点条目前的序号会跳回正文中该链接出现的位置。
- 列表页 / 标签页 / 归档：左栏标签云，右栏近期笔记 + 归档 + 系列。
- 「相关笔记」按 `hugo.yaml` 的 `related` 配置算相似度（标签权重 100、系列 60），不用手工维护。

改侧栏时注意：PaperMod 的 `baseof.html` 用 `partialCached "footer.html" . .Layout .Kind ...` 缓存页脚，
所以 `extend_footer.html` 里**只能放同一 Kind 下所有页面都一样的内容**，否则会被第一个渲染到的页面腌住。
按页变化的部分（系列、相关笔记）放在 `extend_post_content.html`，那个钩子没有缓存。

## 目录结构
```
hugo.yaml                                 # 站点配置（菜单、KaTeX、搜索、related、giscus 等）
content/
  posts/                                  # 所有论文笔记放这里
  about.md  archives.md  search.md
layouts/partials/extend_head.html         # KaTeX 公式注入
layouts/partials/extend_post_content.html # 右栏按页内容（系列 / 相关笔记）
layouts/partials/extend_footer.html       # 侧栏通用部分 + giscus
assets/css/extended/                      # 排版与侧栏样式
static/                                   # 图片等静态资源
themes/PaperMod/                          # 主题（git submodule）
.github/workflows/hugo.yml                # 自动部署
```
