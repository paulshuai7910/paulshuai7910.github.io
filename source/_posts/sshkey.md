---
title: sshkey 多平台配置
date: 2026-04-22 13:39:25
tags: git gitlab github gitee
---

要在本地同时管理 GitHub 和 GitLab 的 SSH 连接，核心思路是**为不同平台生成独立的密钥对**，并通过 SSH 配置文件 (`config`) 自动匹配。这样在拉取或推送代码时，系统会自动选择对应的密钥，互不干扰。

以下是详细的配置步骤（以 macOS/Linux 为例，Windows 用户请在 Git Bash 中操作）：

### 第一步：生成两对独立的 SSH 密钥

建议使用 `ed25519` 算法（更安全）或 `rsa`。生成时务必使用 `-f` 参数指定不同的文件名，避免覆盖。

打开终端，执行以下命令：

```bash
# 为 GitHub 生成密钥
ssh-keygen -t ed25519 -C "your-email@github.com" -f ~/.ssh/id_ed25519_github

# 为 GitLab 生成密钥
ssh-keygen -t ed25519 -C "your-email@gitlab.com" -f ~/.ssh/id_ed25519_gitlab
```

- **说明**：按回车后，如果提示设置密码（passphrase），可以设置（推荐）或直接留空。完成后会在 `~/.ssh/` 目录下生成 4 个文件：`id_ed25519_github`（私钥）、`id_ed25519_github.pub`（公钥）、`id_ed25519_gitlab`、`id_ed25519_gitlab.pub`。

### 第二步：将公钥分别添加到远程平台

1.  **复制公钥内容**：
    ```bash
    # 查看并复制 GitHub 公钥
    cat ~/.ssh/id_ed25519_github.pub
    # 查看并复制 GitLab 公钥
    cat ~/.ssh/id_ed25519_gitlab.pub
    ```
2.  **添加公钥**：
    - **GitHub**：登录 GitHub -> Settings -> **SSH and GPG keys** -> **New SSH key** -> 粘贴 `id_ed25519_github.pub` 的内容。
    - **GitLab**：登录 GitLab -> 点击头像 -> **Preferences** -> **SSH Keys** -> 粘贴 `id_ed25519_gitlab.pub` 的内容。

### 第三步：配置 SSH Config 文件（关键）

在 `~/.ssh/` 目录下创建或编辑 `config` 文件，告诉 SSH 客户端“访问哪个域名用哪把钥匙”。

```bash
# 编辑配置文件
vim ~/.ssh/config
```

**文件内容示例**：

```ini
# GitHub 配置
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github
    IdentitiesOnly yes

# GitLab 配置
Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/id_ed25519_gitlab
    IdentitiesOnly yes
```

- **参数解释**：
  - `Host`：匹配的域名（即你 clone 地址中的 `git@` 后面的部分）。
  - `IdentityFile`：指定该域名使用的私钥路径。
  - `IdentitiesOnly yes`：强制只使用配置文件中指定的密钥，不尝试默认密钥，避免混淆。

### 第四步：测试连接与权限

配置完成后，测试连接是否成功：

```bash
# 测试 GitHub
ssh -T git@github.com
# 成功会显示：Hi your-username! You've successfully authenticated...

# 测试 GitLab
ssh -T git@gitlab.com
# 成功会显示：Welcome to GitLab, @your-username!
```

如果出现 `Permission denied (publickey)` 错误，请检查第 2 步公钥是否添加正确，或尝试重启 SSH 代理：`eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519_github`。

### 第五步：项目级 Git 用户配置（提交记录隔离）

SSH 只负责“通道”认证，提交代码时的作者信息（Commit Author）由 Git 配置决定。为了避免用公司邮箱提交到个人项目，建议在项目目录内单独设置：

```bash
# 进入项目根目录
cd /path/to/your-project

# 设置该项目的专属用户信息
git config user.name "Your Name"
git config user.email "your-email@domain.com"
```

- **全局配置**：如果你大部分项目都用同一个身份（如 GitHub），可以设置全局默认值：`git config --global user.name "Global Name"`，然后在个别项目（如 GitLab 项目）中执行上述局部配置进行覆盖。

### 常见问题排查

- **克隆现有项目**：如果项目之前是 HTTPS 方式克隆的，需要修改远程地址为 SSH：
  ```bash
  git remote set-url origin git@github.com:username/repo.git
  ```
- **权限错误**：运行 `ssh -vT git@github.com` 查看详细日志，通常能发现是哪个环节的密钥没被正确加载。
- **Windows 路径**：Windows 用户密钥通常存放在 `C:\Users\YourUserName\.ssh\`，配置文件路径同理。

按照以上步骤操作，你的本地环境就能清晰地区分 GitHub 和 GitLab 的 SSH 身份了。

## 首次pull 平台项目错误提示

```sh
clone git@gitee.com:muzhi7/thinklogparser-web.git thinklogparse
Cloning into 'thinklogparse'...
The authenticity of host 'gitee.com (180.76.199.13)' can't be established.
ED25519 key fingerprint is SHA256:+ULzij2u99B9eWYFTw1Q4ErYG/aepHLbu96PAUCoV88.
This key is not known by any other names.
```

核对一下 `known_hosts` 是否已有 `gitee.com` 记录，再结合你的 `~/.ssh/config` 解释原因，这样能明确为什么首次连接会出现那条提示。

这是正常现象，和你的 SSH key 配置本身不冲突。

原因是：

1. ~/.ssh/config 只负责告诉 SSH “连到哪个域名时用哪把私钥”，不负责“信任这个主机是谁”。
2. 你第一次用 SSH 连接 gitee.com 时，本机的 ~/.ssh/known_hosts 里还没有 gitee.com 的主机公钥记录。
3. 所以 OpenSSH 会提示：
   The authenticity of host ... can't be established.
   让你确认一次，防止中间人攻击。
4. 你输入 yes 后，主机公钥被写入 ~/.ssh/known_hosts，后续再连同一个主机通常就不会再提示了。

你这次日志也印证了这一点：

- 当时提示 unknown host
- 你确认 yes
- 随后显示 Permanently added gitee.com
- 现在我检查到 known_hosts 已有 gitee.com 记录

补充：
如果未来再次出现同类提示，常见是以下情况之一：

1. 换了新机器或清空了 known_hosts
2. 用了新的主机名/别名连接（例如 IP、不同域名）
3. 远端平台更换了主机密钥（这时要谨慎核对官方指纹后再接受）
