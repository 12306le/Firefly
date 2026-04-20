---
title: 白嫖 GitHub Actions 当定时任务平台
published: 2026-04-20
description: 不想租服务器又想跑定时脚本？GitHub Actions 的 cron 触发器够用了，但有几个坑得知道。
tags: [GitHub Actions, 自动化, cron]
category: 折腾日记
draft: false
---

我之前有几个小需求：每天爬一次某个网站看有没有新内容、每周自动发一次提醒、每小时备份一下配置。最早是开了个最便宜的 VPS 跑 crontab，后来发现 GitHub Actions 直接带 cron 触发器，Public 仓库完全免费，就慢慢全迁过去了。

## 最小可用的工作流长这样

```yaml
name: Daily Task

on:
  schedule:
    - cron: "17 3 * * *"
  workflow_dispatch:

jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "hello at $(date)"
```

`workflow_dispatch` 那行很重要，它能让你在 Actions 页面手动触发一次，调试时不用等定时触发。

## 几个容易踩的坑

**cron 用的是 UTC 时间**。我一开始写 `0 9 * * *` 想着早上九点跑，结果是北京时间下午五点。加 8 小时换算。

**不要用整点**。官方文档都直说了：大量用户都把任务定在 `:00`，GitHub 的调度队列会挤，延迟十几分钟是常事。随手偏移几分钟，比如 `17 3 * * *`（3:17），命中率明显高。

**定时任务会被静默禁用**。如果仓库连续 60 天没活动，Actions 会停掉所有 schedule 触发器。随便推个空提交或者改个 README 就能续命，或者写个 Action 每周自动给自己推一个空提交（虽然有点自欺欺人）。

**私有仓库消耗免费额度**。Public 仓库免费分钟数无限，Private 仓库个人账户每月 2000 分钟。轻量任务够用，但跑构建、跑爬虫的话要算一下。

## 存敏感信息用 Secrets

需要 API Key、Cookie、Token 这种东西，千万别写进 yml，扔到仓库 Settings → Secrets and variables → Actions 里，用的时候 `${{ secrets.XXX }}` 引用。日志里会自动打码成 `***`。

## 最后一个小技巧

Actions 跑失败了默认会发邮件通知，但邮件容易被折叠。我一般再加一步，失败时往 Telegram 或者自建的 Webhook 推一条消息，这样不容易漏掉。

```yaml
      - name: Notify on failure
        if: failure()
        run: |
          curl -s -X POST "https://api.telegram.org/bot${{ secrets.TG_TOKEN }}/sendMessage" \
            -d chat_id=${{ secrets.TG_CHAT_ID }} \
            -d text="Action failed: ${{ github.workflow }}"
```

白嫖归白嫖，该写的错误处理还是得写。
