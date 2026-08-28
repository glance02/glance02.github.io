本文旨在租用新的autodl服务器时可以快速地安装一些常用的工具。主要是在 ubuntu 
服务器上。

可以先打开学术加速：

```bash
source /etc/network_turbo
```

## nvim

nvim 安装和配置，参考 [在AutoDL上配置nvim](../编辑器/在AutoDL上配置nvim.md)

## yazi

### 安装yazi

Yazi 现在已经有官方 Ubuntu/Debian APT 仓库，推荐直接把官方源加进去进行安装。

```bash
apt install -y curl ca-certificates file

curl -fsSL https://yazi-rs.github.io/builds/yazi-keyring.gpg \
  | tee /usr/share/keyrings/yazi-keyring.gpg >/dev/null

echo 'deb [signed-by=/usr/share/keyrings/yazi-keyring.gpg] https://yazi-rs.github.io/builds/ stable main' \
  | tee /etc/apt/sources.list.d/yazi.list >/dev/null

apt update

apt install yazi
```

### 配置 y （实现文件跳转）

主要就在fish中使用y。Fish 推荐一种更干净的方式：单独给函数建文件：

```bash
mkdir -p ~/.config/fish/functions
nvim ~/.config/fish/functions/y.fish
```

配置代码：

```
function y
	set tmp (mktemp -t "yazi-cwd.XXXXXX")
	command yazi $argv --cwd-file="$tmp"
	if read -z cwd < "$tmp"; and [ "$cwd" != "$PWD" ]; and test -d "$cwd"
		builtin cd -- "$cwd"
	end
	command rm -f -- "$tmp"
end
```

会自动加载，不用刷新配置。

## fish

可以直接安装。

```bash
apt install fish
```

版本较旧，直接使用默认主题即可

## zellij

### 安装zellij

AutoDL 旧 Ubuntu 环境，建议直接装 **musl 版本**，这样基本不受旧版 `glibc` 影响：

```bash
cd /tmp

curl -L \
  https://github.com/zellij-org/zellij/releases/latest/download/zellij-x86_64-unknown-linux-musl.tar.gz \
  -o zellij.tar.gz

tar -xzf zellij.tar.gz

install -m 755 zellij /usr/local/bin/zellij
```

### 配置主题

配置一个简单的主题就可以了。

修改配置文件：

```bash
nvim ~/.config/zellij/config.kdl
```

查找 theme，然后把值替换成喜欢的，比如 `catppuccin-mocha`


## 关闭学术加速

```
unset http_proxy && unset https_proxy
```