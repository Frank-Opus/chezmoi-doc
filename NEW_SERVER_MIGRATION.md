# 新服务器安装与迁移指南

这份文档用于在一台新的 Ubuntu 服务器上安装当前这套终端工具，并从 `chezmoi` 仓库恢复已托管的配置。

当前 `chezmoi` 仓库地址：

```bash
https://github.com/Frank-Opus/chezmoi-doc.git
```

## 1. 基础依赖

先安装基础工具：

```bash
apt-get update
apt-get install -y curl git unzip
mkdir -p ~/.local/bin
```

## 2. 安装 starship 和 chezmoi

安装 `starship`：

```bash
curl -fsSL https://starship.rs/install.sh | sh -s -- -y -b ~/.local/bin
```

安装 `chezmoi`：

```bash
sh -c "$(curl -fsLS get.chezmoi.io)" -- -b ~/.local/bin
```

## 3. 安装终端工具

先安装 Ubuntu 仓库里的工具：

```bash
apt-get update
apt-get install -y ripgrep zoxide btop duf bat fd-find fzf unzip
```

修正 `bat` 和 `fd` 的命令名：

```bash
ln -sf /usr/bin/batcat ~/.local/bin/bat
ln -sf /usr/bin/fdfind ~/.local/bin/fd
```

安装 GitHub release 版本的工具：

```bash
tmpdir=$(mktemp -d)
cd "$tmpdir"

curl -fLO https://github.com/zellij-org/zellij/releases/download/v0.44.1/zellij-x86_64-unknown-linux-musl.tar.gz
tar -xzf zellij-x86_64-unknown-linux-musl.tar.gz
install -m 0755 zellij ~/.local/bin/zellij

curl -fLO https://github.com/jesseduffield/lazygit/releases/download/v0.61.1/lazygit_0.61.1_linux_x86_64.tar.gz
tar -xzf lazygit_0.61.1_linux_x86_64.tar.gz
install -m 0755 lazygit ~/.local/bin/lazygit

curl -fLO https://github.com/jesseduffield/lazydocker/releases/download/v0.25.2/lazydocker_0.25.2_Linux_x86_64.tar.gz
tar -xzf lazydocker_0.25.2_Linux_x86_64.tar.gz
install -m 0755 lazydocker ~/.local/bin/lazydocker

curl -fLO https://github.com/eza-community/eza/releases/download/v0.23.4/eza_x86_64-unknown-linux-gnu.tar.gz
tar -xzf eza_x86_64-unknown-linux-gnu.tar.gz
install -m 0755 eza ~/.local/bin/eza

curl -fLO https://github.com/sxyazi/yazi/releases/download/v26.1.22/yazi-x86_64-unknown-linux-musl.zip
unzip -q yazi-x86_64-unknown-linux-musl.zip -d yazi-extract
install -m 0755 yazi-extract/*/yazi ~/.local/bin/yazi
install -m 0755 yazi-extract/*/ya ~/.local/bin/ya

curl -fLO https://github.com/fastfetch-cli/fastfetch/releases/download/2.61.0/fastfetch-linux-amd64.deb
apt-get install -y ./fastfetch-linux-amd64.deb

cd /
rm -rf "$tmpdir"
```

## 4. 拉取并应用 chezmoi 配置

初始化并应用配置：

```bash
chezmoi init https://github.com/Frank-Opus/chezmoi-doc.git
chezmoi apply
```

重新进入 shell，让提示符和跳转命令生效：

```bash
exec bash
```

## 5. 已经会自动恢复的配置

当前仓库里已经托管这些内容：

- `~/.bashrc`
- `~/.bash_logout`
- `~/.profile`
- `~/.gitconfig`
- `~/.config/git/ignore`
- `~/.config/gh/config.yml`
- `~/.config/fish/fish_variables`
- `~/.config/systemd/user/hermes-gateway.service`

## 6. 还需要手工迁移的敏感内容

下面这些内容目前没有放进仓库，迁移新服务器时需要手工处理：

- `~/.config/fish/config.fish`
  里面有 API Key，不适合明文放到 GitHub
- `~/.config/gh/hosts.yml`
  里面通常有 GitHub 登录 token
- `~/.ssh/authorized_keys`
- `~/.ssh/known_hosts`
- `~/.docker/.token_seed`

如果以后要让这些也能自动同步，建议下一步给 `chezmoi` 配置加密，再把敏感文件加进去。

## 7. 安装后验证

执行下面的命令确认环境恢复正常：

```bash
starship --version
chezmoi --version
zellij --version
rg --version
zoxide --version
lazygit --version
lazydocker --version
eza --version
yazi --version
btop --version
duf --version
bat --version
fastfetch --version
fzf --version
fd --version
```

再执行：

```bash
chezmoi managed
chezmoi status
```

如果 `chezmoi status` 没有异常，说明配置已经成功恢复。
