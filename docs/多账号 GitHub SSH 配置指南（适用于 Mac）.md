# **多账号 GitHub SSH 配置指南（适用于 Mac）**

在实际开发中，你可能会同时拥有多个 GitHub 账号（例如工作账号和个人账号），为了避免身份混淆、SSH 密钥冲突、push 被拒等问题，本文将提供一套稳定且清晰的多账号配置方案。

------



## **✅ 目标**

- 为每个 GitHub 账号创建独立的 SSH 密钥
- 将不同仓库与对应密钥绑定
- 自动切换 Git 用户名与邮箱
- 不同项目无需手动切换任何配置

------



## **🧱 文件结构（示例）**

```
~/
├── .ssh/
│   ├── id_ed25519_swiftuihome
│   ├── id_ed25519_swiftuihome.pub
│   ├── id_ed25519_wumacms
│   ├── id_ed25519_wumacms.pub
│   └── config
├── .gitconfig
├── .gitconfig-swiftuihome
└── .gitconfig-wumacms
```

------



## **✍️ 第一步：为每个账号生成 SSH 密钥**

```sh
ssh-keygen -t ed25519 -C "swiftuihome@gmail.com" -f ~/.ssh/id_ed25519_swiftuihome
ssh-keygen -t ed25519 -C "wumamail@qq.com" -f ~/.ssh/id_ed25519_wumacms
```

------



## **🔐 第二步：将公钥添加到对应 GitHub 账号**

登录 GitHub：

- 打开个人账号，进入 [**Settings > SSH and GPG keys**](https://github.com/settings/keys)
- 添加 ~/.ssh/id_ed25519_swiftuihome.pub 内容
- 切换到工作账号，重复步骤添加 id_ed25519_wumacms.pub

------



## **⚙️ 第三步：配置 SSH 主机别名（~/.ssh/config）**

```ini
# ~/.ssh/config

# swiftuihome 账号配置
Host github.com-swiftuihome
    HostName ssh.github.com
    Port 443
    User git
    IdentityFile ~/.ssh/id_ed25519_swiftuihome
    IdentitiesOnly yes

# wumacms 账号配置
Host github.com-wumacms
    HostName ssh.github.com
    Port 443
    User git
    IdentityFile ~/.ssh/id_ed25519_wumacms
    IdentitiesOnly yes
```

------



## **🧩 第四步：配置全局 Git 文件（~/.gitconfig）**

```ini
# ~/.gitconfig

[user]
    name = zeldafox
    email = zeldafoxmail@gmail.com

[filter "lfs"]
    process = git-lfs filter-process
    required = true
    clean = git-lfs clean -- %f
    smudge = git-lfs smudge -- %f

# 项目路径匹配规则（目录名结尾一定要加 /）
[includeIf "gitdir:/Users/devlink/code/github/swiftuihome/"]
    path = ~/.gitconfig-swiftuihome

[includeIf "gitdir:/Users/devlink/code/github/wumacms/"]
    path = ~/.gitconfig-wumacms
```

------



## **🧾 第五步：每个账号独立 Git 配置文件**

### **1️⃣** **~/.gitconfig-swiftuihome**

```ini
# ~/.gitconfig-swiftuihome
[user]
    name = swiftuihome
    email = zeldafox@gmail.com

[core]
    sshCommand = ssh -i ~/.ssh/id_ed25519_swiftuihome
```



### **2️⃣** **~/.gitconfig-wumacms**

```ini
# ~/.gitconfig-wumacms
[user]
    name = wumacms
    email = wumamail@qq.com

[core]
    sshCommand = ssh -i ~/.ssh/id_ed25519_wumacms
```

这些文件会在 Git 仓库路径位于指定目录下时自动生效。

------



## **📦 第六步：克隆或设置 Git 仓库**

请注意不要使用默认地址（如 github.com:），而要使用上面定义的 Host 别名：

```sh
# 克隆 swiftuihome 项目
git clone git@github.com-swiftuihome:swiftuihome/TailwindButtonKit.git

# 克隆 wumacms 项目
git clone git@github.com-wumacms:wumacms/SwiftUIButtonKit.git
```

或者修改现有仓库的 remote：

```sh
git remote set-url origin git@github.com-swiftuihome:swiftuihome/TailwindButtonKit.git
```

------



## **🧪 第七步：验证 SSH 连接与 Git 用户**

```sh
ssh -T git@github.com-swiftuihome
ssh -T git@github.com-wumacms

cd ~/code/github/swiftuihome/TailwindButtonKit
git config user.name   # 应显示 swiftuihome
git config user.email  # 应显示 zeldafox@gmail.com
```

------



## **🧠 小技巧：避免 remote 错误时身份错乱**

你也可以在每个项目的 .git/config 文件中加上：

```ini
[core]
    sshCommand = ssh -i ~/.ssh/id_ed25519_XXX
```

或者直接在项目根目录执行（不推荐长期维护）：

```sh
git config core.sshCommand "ssh -i ~/.ssh/id_ed25519_swiftuihome"
```

------



## **✅ 最终效果**

| **项目路径**               | **Git 用户名** | **Git 邮箱**       | **SSH 私钥**                  | **Git remote Host**    |
| -------------------------- | -------------- | ------------------ | ----------------------------- | ---------------------- |
| ~/code/github/swiftuihome/ | swiftuihome    | zeldafox@gmail.com | ~/.ssh/id_ed25519_swiftuihome | github.com-swiftuihome |
| ~/code/github/wumacms/     | wumacms        | wumamail@qq.com    | ~/.ssh/id_ed25519_wumacms     | github.com-wumacms     |

------

