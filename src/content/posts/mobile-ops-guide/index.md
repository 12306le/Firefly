---
title: 手机党运维速查手册：tmux + Claude Code + Ubuntu 三合一
published: 2026-04-20
description: "专为用 Termius 连服务器的手机党写的速查手册：tmux 防断线保任务、Claude Code 换 API（anyrouter 镜像）、Ubuntu 基本操作，一篇全搞定。"
tags: ["tmux", "Claude Code", "Ubuntu", "Termius", "anyrouter", "速查手册"]
category: 运维指南
pinned: true
draft: false
---

> **写在前面**：手机屏幕小、输入慢，SSH 一断开任务就丢，命令还老记不住。这篇文章就是给手机 Termius 党准备的"贴身小抄"——三大块内容、可直接复制的命令、出问题就翻得到答案。

## 📑 目录导航

- [Part 1 · tmux 防断线保任务](#part-1--tmux-防断线保任务)
  - [为什么要用 tmux](#为什么要用-tmux)
  - [最常用的 6 个命令](#最常用的-6-个命令)
  - [快捷键速查表](#tmux-快捷键速查表)
  - [手机党最佳实践](#手机党最佳实践)
- [Part 2 · Claude Code 换 API（anyrouter 镜像）](#part-2--claude-code-换-apianyrouter-镜像)
  - [为什么要换 API](#为什么要换-api)
  - [anyrouter 注册与获取 Key](#anyrouter-注册与获取-key)
  - [配置环境变量](#配置环境变量)
  - [验证与常见问题](#验证与常见问题)
- [Part 3 · Ubuntu 基本操作速查](#part-3--ubuntu-基本操作速查)
  - [文件与目录](#文件与目录)
  - [软件安装](#软件安装)
  - [进程与服务](#进程与服务)
  - [网络与端口](#网络与端口)
  - [磁盘与权限](#磁盘与权限)

---

## Part 1 · tmux 防断线保任务

### 为什么要用 tmux

手机 Termius 连服务器最大的痛点：

- 🚌 坐地铁信号差 → SSH 断线 → **任务直接没了**
- 🔋 手机灭屏久了 → 连接超时 → **前面跑了半小时的脚本白跑**
- 📞 切到微信回消息 → 回来一看终端已断开

**tmux 就是解药**：它把你的终端跑在服务器后台，SSH 断不断开和它无关。下次连上服务器，一条命令就回到原来的现场，任务还在跑。

### 最常用的 6 个命令

只要记住这 6 条，80% 场景够用：

```bash
# 1. 新建一个窗口叫 work（第一次用就这样）
tmux new -s work

# 2. 先暂时离开（任务继续在后台跑），按 Ctrl+b 松开，再按 d
# 或者直接关掉 Termius 也行，tmux 不会死

# 3. 回来继续干活
tmux a -t work

# 4. 忘了叫什么名字？列出所有会话
tmux ls

# 5. 干完活了，彻底关掉这个会话
tmux kill-session -t work

# 6. 一键关掉所有 tmux
tmux kill-server
```

### tmux 快捷键速查表

tmux 的所有快捷键都要**先按前缀键 `Ctrl+b`**，松开后再按下一个键。

| 按键 | 作用 | 场景 |
|------|------|------|
| `Ctrl+b` `d` | 离开当前会话（detach） | 要下班了，任务继续跑 |
| `Ctrl+b` `c` | 新建窗口 | 想同时跑俩命令 |
| `Ctrl+b` `n` / `p` | 下一个 / 上一个窗口 | 切换窗口 |
| `Ctrl+b` `数字` | 跳到指定窗口 | 如 `Ctrl+b 0` 跳到 0 号窗口 |
| `Ctrl+b` `,` | 给窗口改名 | 多窗口时方便识别 |
| `Ctrl+b` `%` | 左右分屏 | 一边跑程序一边看日志 |
| `Ctrl+b` `"` | 上下分屏 | 同上 |
| `Ctrl+b` `o` | 在分屏之间切换 | |
| `Ctrl+b` `x` | 关掉当前分屏 | |
| `Ctrl+b` `[` | 进入滚屏模式（按 q 退出） | 翻看上面的输出 |

> 💡 **手机 Termius 小贴士**：Termius 键盘上有专门的 `Ctrl` 键，先点 `Ctrl` 再点 `b` 就是前缀键。

### 手机党最佳实践

**推荐工作流程**：

```bash
# 一上服务器就进 tmux
tmux a -t work 2>/dev/null || tmux new -s work
```

这条命令的意思是：**有就进，没有就新建**。把它写进 `~/.bashrc` 最后一行，每次 SSH 一连就自动进 tmux：

```bash
echo '[[ -z "$TMUX" ]] && (tmux a -t work 2>/dev/null || tmux new -s work)' >> ~/.bashrc
```

> ⚠️ 注意：别对 root 或脚本化的 ssh 会话用这招，不然会循环嵌套。

**救命三连（手机刚断线回来）**：

```bash
tmux ls          # 先看还有哪些会话
tmux a -t work   # 接回原来那个
# 如果有多个会话，attach 到最近的：
tmux a
```

---

## Part 2 · Claude Code 换 API（anyrouter 镜像）

### 为什么要换 API

官方 Anthropic API 在国内直连**不稳定**，而且需要海外支付方式。**anyrouter 镜像**（<https://anyrouter.top>）解决了：

- ✅ 国内直连稳定，不用代理
- ✅ 支持支付宝/微信充值
- ✅ 兼容官方 API，Claude Code 改一个环境变量就能用

### anyrouter 注册与获取 Key

1. 打开 <https://anyrouter.top>
2. 手机号或邮箱注册登录
3. 进入"**令牌 / API Key**"页面，点"**添加新令牌**"
4. 复制出来的 Key，形如 `sk-xxxxxxxxxxxx...`
5. 充值余额（看着用，一般 10 块够玩好久）

### 配置环境变量

Claude Code 认两个环境变量：

| 变量名 | 填什么 |
|--------|--------|
| `ANTHROPIC_BASE_URL` | `https://anyrouter.top` |
| `ANTHROPIC_AUTH_TOKEN` | 第 4 步复制的 Key |

**一次性生效（只对当前终端有效）**：

```bash
export ANTHROPIC_BASE_URL="https://anyrouter.top"
export ANTHROPIC_AUTH_TOKEN="sk-你的Key"
claude
```

**永久生效（推荐）**：写入 `~/.bashrc`

```bash
cat >> ~/.bashrc <<'EOF'

# Claude Code via anyrouter
export ANTHROPIC_BASE_URL="https://anyrouter.top"
export ANTHROPIC_AUTH_TOKEN="sk-你的Key"
EOF

source ~/.bashrc
```

> ⚠️ **关键坑点**：如果你之前设置过 `ANTHROPIC_API_KEY`，**必须先清掉**，不然 Claude Code 会优先用它而不走镜像：
>
> ```bash
> unset ANTHROPIC_API_KEY
> ```
>
> 要彻底干净，临时启动也可以用 `env -u`：
>
> ```bash
> env -u ANTHROPIC_API_KEY claude
> ```

### 验证与常见问题

**验证是否生效**：

```bash
echo $ANTHROPIC_BASE_URL   # 应该输出 https://anyrouter.top
claude --version           # 看看是否正常启动
```

**常见问题**：

| 报错 | 原因 | 解决 |
|------|------|------|
| `401 Unauthorized` | Key 错了 / 没改环境变量 | 重新复制 Key，`source ~/.bashrc` |
| `余额不足` | 欠费了 | 回 anyrouter 充值 |
| 一直连官方 API | `ANTHROPIC_API_KEY` 没清 | `unset ANTHROPIC_API_KEY` |
| 速度慢 | 高峰期 | 换时段 or 升级套餐 |

---

## Part 3 · Ubuntu 基本操作速查

> 手机 Termius 输入慢，下面的命令都是**最短最实用**的版本，直接复制改参数就能用。

### 文件与目录

```bash
pwd                      # 我在哪
ls -lh                   # 列当前目录（-h 是人类可读的文件大小）
ls -lha                  # 带隐藏文件
cd ~                     # 回家目录
cd -                     # 回上一次的目录

mkdir -p a/b/c           # 一口气建多层目录
touch foo.txt            # 建空文件

cp file1 file2           # 复制文件
cp -r dir1 dir2          # 复制目录（-r 递归）
mv a b                   # 移动 / 改名
rm file                  # 删文件
rm -rf dir               # 删目录（⚠️ 不可恢复，看清路径再按回车）

cat file                 # 看整个文件
less file                # 分页看（q 退出，/关键词 搜索）
head -20 file            # 看前 20 行
tail -f log              # 实时追加看日志（Ctrl+c 退出）

find . -name "*.log"     # 按文件名找
grep -rn "关键词" .      # 在当前目录递归搜内容
```

### 软件安装

Ubuntu 用 `apt`：

```bash
sudo apt update                          # 先刷软件源（必做）
sudo apt install -y curl git tmux htop   # 装常用工具，-y 是省掉确认

sudo apt upgrade -y                      # 升级已装的软件
sudo apt remove 包名                     # 卸载
sudo apt autoremove                      # 清掉不再需要的依赖
```

装 Node.js / Python / Docker 这些大件推荐用官方脚本，别用 apt 的老版本。

### 进程与服务

```bash
# 看进程
ps aux | grep nginx        # 搜特定进程
top                        # 实时进程监视（按 q 退出）
htop                       # 更好看的 top（需要 apt install htop）

# 杀进程
kill PID                   # 正常关
kill -9 PID                # 强杀（杀不掉的终极手段）
pkill -f 关键词            # 按名字杀一批

# 服务管理（systemd）
sudo systemctl status nginx    # 看状态
sudo systemctl start nginx     # 启动
sudo systemctl stop nginx      # 停止
sudo systemctl restart nginx   # 重启
sudo systemctl enable nginx    # 开机自启
```

### 网络与端口

```bash
curl -I https://google.com   # 看响应头，测试连通性
ping -c 4 baidu.com          # ping 4 次就停

# 端口占用（查 80 端口被谁占了）
sudo ss -tlnp | grep :80
# 或者老办法
sudo lsof -i :80

# 放行端口（ufw 防火墙）
sudo ufw allow 80
sudo ufw status
```

### 磁盘与权限

```bash
df -h                # 看磁盘剩多少（-h 人类可读）
du -sh *             # 当前目录下每一项占多少
du -sh * | sort -h   # 从小到大排序
free -h              # 看内存

# 权限
chmod +x script.sh   # 加可执行权限
chmod 755 file       # rwxr-xr-x
chown user:user file # 改属主

# 我是谁 / 我属于谁
whoami
groups
id
```

---

## 🎒 带走这张速查卡

**手机党三条金律**：

1. **一上服务器先进 tmux**：`tmux a -t work || tmux new -s work`
2. **Claude Code 不通就检查环境变量**：`echo $ANTHROPIC_BASE_URL` 和 `unset ANTHROPIC_API_KEY`
3. **任何 `rm -rf` 之前多看一眼路径**，手机误触代价很大

收藏这篇文章，手机断网回来翻一翻，什么问题都有答案。 🚀

---

**相关资源**：

- [tmux 官方文档](https://github.com/tmux/tmux/wiki)
- [anyrouter 镜像](https://anyrouter.top)
- [Claude Code 官方文档](https://docs.claude.com/claude-code)
- [Ubuntu 官方手册](https://help.ubuntu.com/)
