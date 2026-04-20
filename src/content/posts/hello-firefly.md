---
title: 开张第一篇 —— 把博客搬到 GitHub Pages
published: 2026-04-20
description: 从 Fork 模板到在线访问，记录一次完整的静态博客部署过程，顺手吐槽踩过的那几个坑。
tags: [博客, GitHub Pages, Astro, Firefly]
category: 折腾日记
draft: false
pinned: true
---

这是这个博客的第一篇文章，随便写点东西占位，顺便说说我是怎么把它弄起来的。

## 为什么用静态博客

以前想过要写博客这件事，挑过 WordPress、Typecho、Halo 之类的动态站，结果每次都败在"得买服务器"这一步。静态博客就不一样了，内容是提前生成好的 HTML，白嫖 GitHub Pages、Vercel、Cloudflare Pages 都能跑，不用管数据库、不用管备份、不用怕被脱库。

当然代价是每次发文章都要重新构建一次站点，但对更新频率不高的个人博客，这点成本完全可以接受。

## 选模板

市面上 Astro 的博客主题挺多的，Fuwari 算是中文圈里比较火的一个，Firefly 是在 Fuwari 基础上二次开发出来的，侧边栏、瀑布流、文章网格这些加得都挺实用。选它主要是看着顺眼，另一方面是文档写得比较细，对新手友好。

## 部署到 GitHub Pages 的小坑

模板自带的 `deploy.yml` 会把构建产物推到 `pages` 分支，设置里把 Pages 的源切到这个分支就能出站点。有几个点要注意：

- **`base` 路径**：用户名仓库（`xxx.github.io`）可以保持 `/`，项目仓库（像这种 `xxx.github.io/Firefly`）必须改成 `/Firefly/`，不然所有静态资源都会 404。
- **`site_url`**：`siteConfig.ts` 里那一行要改成自己的实际域名，RSS、sitemap、分享海报都会引用它，不改的话生成出来的链接全是原作者的站点。
- **别忘了删原作者的统计 ID**：Google Analytics 和 Microsoft Clarity 的 ID 如果不清掉，访客数据会稀里糊涂上传给别人。

## 接下来

这博客主要会放点自己折腾过程中的笔记，命令行工具、部署经验、小白视角的踩坑实录这类。更新频率随缘，图个开心。
