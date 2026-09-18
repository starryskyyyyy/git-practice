# Git 速查表

> 每天开工先 `git pull`，写完代码 `git status` → `git add` → `git commit` → `git push`。

## 日常循环（90% 的时间只用这几条）

| 命令 | 作用 |
|---|---|
| `git status` | 看当前状态。**出错时第一件事就是敲它** |
| `git diff` | 看具体改了什么内容（`+` 新增，`-` 删除） |
| `git add .` | 把所有改动放进暂存区 |
| `git add <文件>` | 只把某个文件放进暂存区 |
| `git commit -m "说明"` | 存一个档 |
| `git push` | 推到 GitHub |
| `git pull` | 从 GitHub 拉最新（**每天开工第一件事**） |
| `git log --oneline` | 看存档历史，一行一个 |

## 开始一个项目

| 命令 | 作用 |
|---|---|
| `git init` | 把当前文件夹变成 Git 仓库 |
| `git clone <url>` | 把 GitHub 上的项目下载到本地 |
| `git remote add origin <url>` | 关联远程仓库 |
| `git push -u origin main` | 第一次推送并建立追踪（之后就只敲 `git push`） |

## 分支

| 命令 | 作用 |
|---|---|
| `git branch` | 列出所有分支 |
| `git switch -c <名字>` | 创建并切换到新分支 |
| `git switch <名字>` | 切换分支 |
| `git merge <名字>` | 把某个分支合并到当前分支 |

## 后悔药

| 命令 | 作用 |
|---|---|
| `git restore <文件>` | 丢弃某个文件**未暂存**的改动 |
| `git restore --staged <文件>` | 把文件从暂存区拿出来（改动还在） |
| `git commit --amend` | 修改**最近一次**提交的说明 |
| `git reset --soft HEAD~1` | 撤销最近一次提交，改动保留在暂存区 |

⚠️ **`git reset --hard` 会永久丢弃未提交的改动，不可恢复。新手阶段先别碰。**

## 查看与排查

| 命令 | 作用 |
|---|---|
| `git show HEAD` | 看最近一次提交的详细内容 |
| `git show --stat HEAD` | 看最近一次提交改了哪些文件 |
| `git diff HEAD` | 看已提交的内容与当前工作区的差异 |
| `git ls-remote origin` | 看远程仓库有哪些分支和提交 |
| `git remote -v` | 看当前关联了哪些远程仓库 |

## 配置（一次性）

```bash
git config --global user.name "starryskyyyyy"
git config --global user.email "ljpxjj@163.com"
git config --global init.defaultBranch main
git config --global core.autocrlf true      # Windows 换行符处理
git config --global credential.helper manager
```

查看当前配置：`git config --global --list`

## 三条保命规则

1. **绝不提交密码、私钥、Token。** 一旦提交过，即使后来删掉，历史里仍能查到。
   `.gitignore` 里提前写：`.env`、`*.key`、`id_rsa*`、`*.pem`
2. **commit 说明写"做了什么"，不要写 "update" / "改了改"。**
   半年后 `git log` 时，这是你唯一的线索。
3. **`git status` 和 `git diff` 是最有用的两个命令。**
   提交前先看一眼，能避开大部分事故。

## 常见报错

| 报错 | 原因 | 解决 |
|---|---|---|
| `Connection timed out` (port 22) | GitHub 22 端口被墙 | 配置 `~/.ssh/config` 走 `ssh.github.com:443` |
| `Connection reset by ... port 443` | GFW 干扰 SSH 协议特征，链路时通时不通 | 见下方「网络问题排查」；根本办法是走代理或换网络 |
| `rejected - fetch first` | 远程有本地没有的提交（通常是建仓库时勾了 README） | 远程是空仓库就 `git push -f`；有真实内容则 `git pull --rebase origin main` 再 push |
| `Permission denied (publickey)` | 公钥没加到 GitHub，或密钥不对 | 检查 `ssh -T git@github.com` |
| `Could not read from remote repository` | 通常是网络问题，不是权限问题 | 先 `ssh -T git@github.com` 分清是网络还是认证 |
| `LF will be replaced by CRLF` | Windows/Linux 换行符差异 | 正常警告，**不用管** |
| `fatal: not a git repository` | 当前目录不是 Git 仓库 | `cd` 到项目目录，或 `git init` |
| `CreateProcessW failed` / `No such file` | SSH 配置里路径用了反斜杠被转义 | ProxyCommand 里的路径改用**正斜杠** `/` |
| `unknown option '-W'` | `connect.exe` 语法与 OpenSSH 不同 | 用 `-S 代理 %h %p`，host/port 是**位置参数** |
| PowerShell 里 Git 成功却显示 `exit code: 1` | Git 把进度信息输出到 stderr | **看输出内容判断成败**，不看退出码 |

## 网络问题排查（中国大陆必备）

**分层定位，不要瞎猜。** 按顺序执行，看哪一层断掉：

```bash
# 第 1 层：TCP 能通吗
Test-NetConnection github.com -Port 443

# 第 2 层：SSH 协议能握手吗（能通说明配置没问题，是链路抖动）
ssh -T git@github.com

# 第 3 层：看实际生效的 SSH 配置（确认 ProxyCommand 有没有被读进去）
ssh -G github.com | Select-String "proxycommand|hostname|port"
```

**本机当前配置（2026-09 起生效）**

| 项目 | 值 |
|---|---|
| 代理软件 | Clash Verge（`clash-verge` / `verge-mihomo` 进程） |
| 代理端口 | `127.0.0.1:7897`（SOCKS5 + HTTP 混合端口） |
| SSH 适配器 | `C:/Program Files/Git/mingw64/bin/connect.exe` |
| 配置文件 | `C:\Users\starryskyyy\.ssh\config` |
| Git HTTPS 代理 | `http.proxy` / `https.proxy` = `http://127.0.0.1:7897` |

**关键概念**：TCP 通 ≠ SSH 能连。GFW 允许 TCP 握手成功，但在应用层识别出 SSH 特征后发 RST 掐断。
所以 `Test-NetConnection` 显示 `True` 却仍然 `Connection reset` 是正常现象，不是工具骗你。

**排查优先级（从省事到麻烦）**

1. **换网络**——手机热点 / 校园网。最省事，经常直接就好了，而且更快
2. **开代理**——确认 Clash Verge 在运行且节点可用
3. **查端口**——Clash 更新后端口可能变，核对 `~/.ssh/config` 里的 `-S` 参数
4. **临时绕开代理**：`git -c core.sshCommand="ssh -o ProxyCommand=none" push`

**代理关闭时会怎样**：Git 会连不上 GitHub（因为 `connect.exe` 连不到 `127.0.0.1:7897`）。
这不是配置坏了，开代理重试即可。想彻底不依赖代理，就把 `~/.ssh/config` 里的 `ProxyCommand` 那两行删掉。
