## 安装nvim

因为AutoDL上面的apt下载的nvim版本很旧，所以需要下载github上的包。需要先打开学术加速，这个AutoDL有。运行：

```bash
source /etc/network_turbo
```

之后在开始安装：

```bash
cd /tmp
curl -LO https://github.com/neovim/neovim/releases/latest/download/nvim-linux-x86_64.tar.gz
tar xzf nvim-linux-x86_64.tar.gz
mv nvim-linux-x86_64 /opt/nvim
ln -sf /opt/nvim/bin/nvim /usr/local/bin/nvim
```

安装完成之后，可以检查一下nvim版本是否是最新的：

```bash
nvim --version
```


## 安装 Lazyvim 配置

运行：

```bash
mv ~/.config/nvim ~/.config/nvim.bak 2>/dev/null
mv ~/.local/share/nvim ~/.local/share/nvim.bak 2>/dev/null
mv ~/.local/state/nvim ~/.local/state/nvim.bak 2>/dev/null
mv ~/.cache/nvim ~/.cache/nvim.bak 2>/dev/null

git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git

nvim
```

等待安装，即可完成
