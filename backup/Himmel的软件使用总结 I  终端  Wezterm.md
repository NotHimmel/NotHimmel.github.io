 # 终端选择哲学
本文在挑选主力终端软件时，主要基于以下要求：
1. 开源
终端软件会接触到各种密码，如果不是真开源是不敢用的。
2. 性能
终端作为基础软件是要长期开着的，不能也没有必要占用太多性能。
3. 多设备配置同步
支持自定义配置文件的设置，方便多个设备之间共享配置。

# 2025选择现状
❌ Windterm     这篇文章的由来，偶然发现该软件并非真开源，代码4年没更新，release却一直在更新... 
❌ Tabby            占用CPU太高

## [Wezterm](https://wezterm.org/index.html)
最新下载链接: https://bjansen.github.io/scoop-apps/versions/wezterm-nightly/
scoop bucket add versions
scoop install wezterm-nightly

git clone https://github.com/KevinSilvester/wezterm-config.git ~/.config/wezterm
主要配置文件
%USERPROFILE%/.config/wezterm/wezterm.lua
修改
./config/domains.lua   for custom SSH/WSL domains
./config/launch.lua  for preferred shells and its paths

### 前置软件
scoop bucket add nerd-fonts
scoop install JetBrainsMono-NF
scoop install nu

### Windows配置右键菜单
win+r 输入 regedit 打开注册表编辑器，依次展开HKEY_CLASSES_ROOT

底下的 HKEY_CLASSES_ROOT\Directory
底下的 HKEY_CLASSES_ROOT\Directory\Background
底下的 HKEY_CLASSES_ROOT\Directory\Background\shell
新建一个项 wezterm ，在 wezterm 编辑 Icon 数据指向wezterm安装程序图标，编辑 （默认） 数据为菜单名称如 Open Wezterm Here，
然后右键新建一个项command，编辑 （默认） 数据："C:\soft\WezTerm-windows\wezterm-gui" start --no-auto-connect --cwd "%V\\"

### 参考
https://github.com/KevinSilvester/wezterm-config
https://github.com/QianSong1/wezterm-config


# [Alacritty](https://alacritty.org) （未完待续）


# 相关
[scoop](https://scoop.sh)
https://github.com/aspiers/stow

# 参考
https://jyn.dev/how-i-use-my-terminal/
