---
title: 手机用 Termius 连服务器的几个舒服姿势
published: 2026-04-20
description: 在手机上敲命令行其实没那么痛苦，前提是你把 Termius 调教好了。
tags: [Termius, SSH, iOS, Android, 折腾]
category: 工具
draft: false
---

很多人以为"手机上搞服务器"只能看看状态，实际上用 Termius 配合一些快捷方式，纯手机也能写代码、发文章、跑脚本。记几条我自己觉得特别救命的设置。

## 密钥登录，别再背密码

Termius 自带密钥管理，生成一对 Ed25519 密钥，把公钥丢进服务器的 `~/.ssh/authorized_keys`，之后登录就直接进。密码登录这事不光烦，暴露在公网的 22 端口每天能被爆破几千次，日志里一眼看过去全是红的。

顺手把服务器的 `/etc/ssh/sshd_config` 改成 `PasswordAuthentication no`，世界清静。

## Snippets 是宝藏功能

手机键盘是原罪，但 Termius 有个 Snippets，能把常用命令存下来，点一下就执行或者塞到输入框里。我存的几条：

- `sudo journalctl -u 服务名 -f --since "10 min ago"` 看日志
- `df -h && free -h` 看磁盘和内存
- `docker ps -a --format 'table {{.Names}}\t{{.Status}}'` 看容器状态
- `htop` 一键打开

这几条基本覆盖了 80% 的"上来先看看"场景，比每次手打舒服太多。

## Port Forwarding 让内网服务触手可及

Termius 的端口转发是 GUI 化的，不用记 `-L` 参数的顺序。本地起的 Jupyter、家里 NAS 的 Web 面板、服务器上的 Grafana，配好一次之后每次连上自动建隧道，手机浏览器里 `localhost:xxxx` 就能访问。

## 字体和配色

默认字体在小屏上看代码很挤，换成等宽字体（比如 JetBrains Mono）行距拉开一点，瞬间不累眼。配色我一般选 Dracula 或者 Solarized Dark，长时间盯屏幕比高对比度主题舒服。

## 一个小遗憾

Termius 的高级功能要订阅才能同步多设备，免费版只能单机配置。如果你经常在手机和电脑之间切换，要么咬牙订阅，要么手动导出导入配置。我自己只用手机一个设备，所以无所谓。

命令行这事儿在手机上折腾久了会上瘾，不骗你。
