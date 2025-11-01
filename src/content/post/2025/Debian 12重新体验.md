---
title: Debian 12重新体验
description: 无描述
publishDate: 2024-01-31
updatedDate: "2025-11-01"
tags: 
- Linux
- Debian
slug: debian12-on-laptop
---


:::important
二〇二五年八月二十九日更新：目前已升级至Debian 13，部分内容可能过时。

二〇二五年一月八日更新：目前又换到KDE了，因为Discover卡顿的问题已经暂时解决了，见[](<#重新回到KDE>)。
:::

上一次[给笔记本装了Windows 8.1](<#数字难民——Windows 8.1在2024年的现状>)，但是由于这个系统比较老（2013年发布，去年停止支持），所以现在很多软件都不支持了。于是又重新安装了Debian 12，遇到了非常多的坑。目前用下来比较顺手的是MATE桌面环境。

<!-- TEASER_END -->

之前用的是[KDE](https://kde.org)桌面环境，其实这个桌面如果只有桌面的话占用非常低（甚至比[Xfce](https://xfce.org)和MATE还低），但是里面的核心组件——Discover这个类似应用商城的必须组件会占用非常多的内存（200多M），导致用起来非常卡。这也是GNOME，Cinnamon同样存在的问题（`gnome-software`占用在150M左右）。

[LXDE](https://www.lxde.org)和[LXQt](https://lxqt-project.org)则是过于简洁，这两个桌面环境到现在都是不支持笔记本盖上盖子自动睡眠的。Xfce是我第一个尝试的轻量桌面，但是
这个桌面环境对于休眠睡眠等功能支持非常不好，如果设置为合上盖子锁屏打开会跳到桌面然后黑屏再显示锁屏，并且无法正常使用休眠功能（一休眠就黑屏）。对此官网[解释说](https://docs.xfce.org/xfce/xfce4-power-manager/faq#i_have_put_my_computer_into_suspend_or_hibernate_mode_but_the_computer_is_not_waking_up)这是内核的问题（所以为什么只有Xfce有这个问题？）。

所以目前换到了[Mate桌面环境](https://mate-desktop.org/)，一个类似GNOME老版本的桌面。它的默认布局我不是非常喜欢，不过这些面板都是可以编辑的，那个难用的左上角应用菜单可以用发行版预装的Brisk Menu替代。

# 主题美化
这可以说是Linux有趣的一点。目前正在使用的主题是`papirus-icon-theme`和`materia-gtk-theme`这两个主题。在MATE桌面上使用这个主题有一个问题——内置的文件管理器`caja`在切换主题后背景会变为黑色。解决方法很简单：找到编辑——背景和徽标——颜色，找到白色拖至背景即可。

登陆界面的GTK主题要单独设置，建议安装`lightdm-gtk-greeter-settings`然后设置GTK主题和图标（注意要点右下角的保存按钮！）。另一个问题是默认lightdm的登陆界面不会显示用户头像，解决方法是编辑`/etc/lightdm.d/lightdm.conf`，找到下面的内容取消注释：

```ini
greeter-hide-users=false
```

## Qt
由于Gtk与Qt是两个不同的GUI框架，所以要分别使用不同的主题。

在KDE上的设置中有GTK主题设置，但是在MATE，Xfce等桌面没有设置。所以需要手动安装`qt5ct`这个设置工具，并设置变量`QT_QPA_PLATFORMTHEME`为`qt5ct`。

对于*Materia*这个主题有`materia-kde`这个为Qt（KDE主题）适配的版本，这个主题需要Kvantum这个Qt主题工具配置。

## 暗黑模式自动切换
在Windows和iOS等平台上用习惯了自动切换暗黑模式后就想在Linux上用，可惜目前的解决方案虽然能达成目标，但都不是特别令人满意。根据[Arch Linux wiki](https://wiki.archlinux.org/title/Dark_mode_switching)，目前用的解决方案是[Yin-yang](https://github.com/oskarsh/Yin-Yang)，可以自动切换GTK，Kvantom，Qt和图标主题，唯一的问题是无法切换Qt的图标主题。

## 其他
Caja有一个独有的问题：Markdown，HTML等文件的图标显示不正常，解决办法是在首选项找到预览——文本文件，将“图标中显示文字”改为“从不”[^1]。

# 优化
首先是换成清华的Debian源：

```bash
sudo sed -ie 's/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list
```

对于笔记本Debian有一个适用于笔记本的优化工具集合，默认不安装，可以在终端输入以下命令安装：

```bash
sudo tasksel install laptop
```

这些工具里面有一个可以给笔记本省电的`powertop`工具，以下命令可以自动给笔记本调整省电设置：

```bash
sudo powertop --auto-tune
```

但是`powertop`不能开机自动运行，要自动设置需要手动新建一个systemd服务。~~建议安装`tlp`，安装这个之后就不用管了~~。这样Linux可以用3小时多。

```ini
[Unit]
Description=Powertop tunings

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/powertop --auto-tune

[Install]
WantedBy=multi-user.target
```

另外建议安装一个工具

```bash
sudo apt install tlp
```

## 终端
默认的`bash`并不是特别好用，~~建议用`zsh`。其实这个Shell不用装[oh-my-zsh](https://ohmyz.sh)直接用默认的配置也可以很好用，一般只改下面的这些配置：~~。目前已经换成[Fish](https://fishshell.com)了，相比zsh增加了搜索历史记录的功能。

```bash
bindkey -v # 使用vim键位
#...
setopt no_nomatch
source ~/.profile
```

其中最后一个命令建议加入Shell的启动文件中，是让`.profile`文件中的环境变量配置生效。倒数第二个命令是为了解决一个恶心的问题：当命令行出现`*`，`?`等通配符字符时zsh会报错，需要改成`\*`才行，添加这个命令就没有这个问题了。

另外MATE的文件管理器不支持“在此处打开终端”的快捷方式，需要安装`caja-open-terminal`这个插件。

### Fish配置
Fish默认配置还算开箱即用，主题和颜色的配置直接输`fish_config`就能在浏览器中配置，`prompt`目前用的是`Nim`主题。

目前只在主配置文件增加设置了Vi快捷键：

```bash
# ~/.config/fish/config.fish
if status is-interactive
    # Commands to run in interactive sessions can go here
    fish_vi_key_bindings
end
```

### ~~主题&插件~~
~~目前选择的是[zplug](https://github.com/zplug/zplug)作为zsh的插件管理器。这个插件管理器虽然没zinit更新得勤快，并且速度比较慢，但是是内置于Debian软件源的~~。原本是为了安装[powerlevel10k](https://github.com/romkatv/powerlevel10k)这个主题而安装zplug的，换成liquidprompt之后就不用插件管理器了。

```bash
source /usr/share/zplug/init.zsh
zplug romkatv/powerlevel10k, as:theme, depth:1
zplug load # 加载安装的插件
```

#### [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)
```bash
source /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
```

这个插件非常常用，可以实现在Powershell 7上自带的历史记录补全的功能（虽然已经不更新了，其实没必要更新）。

#### [liquidprompt](https://github.com/liquidprompt/liquidprompt)
在Debian的官方仓库中发现的Shell提示符主题。之前我用的是zsh内置的clint主题，也就是这样配置，支持显示Git仓库：

```bash
autoload -Uz promptinit
autoload -Uz vcs_info
promptinit
prompt clint # 使用clint主题
```

不过clint主题还是不够漂亮，而Debian仓库里的powerlevel9k太旧，powerlevel10k也已经停止维护了。然后我在官方仓库发现了`liquidprompt`，这个主题在一个月之前发布了正式版本，并且相对于主流的starship等类似主题非常简洁而实用，许多信息都是在需要的时候才显示。

启用这个主题只需要在终端输入：

```
lp_activate
```

## 护眼模式
这个东西在Gnome和KDE都是内置的，但是在别的桌面就没有了，可以安装`redshift`这个软件实现这个功能，这个软件可以按照时间启用护眼模式。

```bash
sudo apt install redshift redshift-gtk
```

## 蓝牙
Debian MATE虽然默认安装了蓝牙服务`bluez`，但是没有自带蓝牙的图形客户端，因此需要安装`blueman`这个蓝牙图形界面配置工具。

```bash
sudo apt install blueman
```

这样就能在系统托盘方便地管理蓝牙了。

另外，安装蓝牙后你会发现蓝牙每次都是开机就启用了。对于安装了`tlp`的Linux，可以编辑`/etc/tlp.conf`然后找到这一行修改，这样关机前蓝牙的启用状态就会在下一次被应用[^2]。

```ini
RESTORE_DEVICE_STATE_ON_STARTUP=1
```

## Debian版本的软件包

### 打印机驱动
虽然提供Linux版本的打印机驱动的厂家不多，但还是有的，其中Canon甚至还提供了打印机驱动的[Debian软件包版本](https://www.canon.com.cn/supports/download/simsdetail/0101131702.html?modelId=1329&channel=)。

### 其他
目前常用的软件的软件包都列在下面：

* *Joplin* 目前活跃更新的在Sourceforge上有一个，见[这里](https://sourceforge.net/projects/joplin-desktop-linux-package/files/deb-repo/)
* *Waterfox* 在OpenSUSE上

## 支持访问iPad等苹果设备
在Linux上没有iTunes用来访问苹果设备，不过Debian的软件源中默认附带了支持访问苹果设备的库，安装下面这些库之后就可以连接苹果设备了[^3]：。

```
sudo apt install libimobiledevice-utils libimobiledevice-dev libgpod-dev
```

## 默认的通知栏不显示图标
这个问题搞了我半天，引起的原因可能是安装了`mate-indicator-applet`这个包。如果不显示暂时的方法是在顶部面板添加`mate-indicator-applet`这个包提供的“指示器”部件。

但是这个东西和原来的通知栏相比体验并不好：原来通知栏的图标这个部件也有自己的一套，而且这个部件无法接受左键点击，默认左键点击映射为右键，右键则是小部件的选项。还原为之前的设置让我弄了很久，直到看到[这个链接](https://ubuntu-mate.community/t/icon-inconsistency-in-panel-system-tray-icons/24794/3)里面的截图。

里面提到的Ayatana就是实现指示器功能的东西，当MATE桌面检测到Ayatana的服务时就会禁用通知栏的图标，解决方法就是到控制中心——启动应用程序，将那堆名称为*Ayatana Indicator*的启动项取消选择，重启。

## 显示Unicode CJK拓展区的字符
这个需求并不是特别重要

1. 安装`fonts-babelstone-han`这个包
2. 到<http://cheonhyeong.com/Simplified/download.html>下载天衍全字库

# 软件

## ~~Syncthing~~
曾经在用的文件同步工具，现在不用了。

## mpv
目前只需这些配置：

```ini
audio-pitch-correction=no
sub-font='LXGW WenKai'
no-keepaspect-window
```

## aria2
aria2是一个多线程下载工具，目前只要用配置改三个设置就可以体会到极速下载速度了。

```ini
max-concurrent-downloads=16
max-connection-per-server=16
min-split-size=1M
```

之前用的是[Motrix](https://motrix.app)作为Aria2的UI，但是Electron占用过大，所以换成了[Aria2ng](https://ariang.mayswind.net/zh_Hans/)，通过`rclone`作为服务器将其作为浏览器应用使用。

## Watt Toolkit代理
使用Watt toolkit这个工具可以加速GitHub､vercel和一些CDN资源网站的访问。目前watt toolkit只有2.x.x版本才提供deb格式的安装包。由于在我电脑上443端口似乎被`fcitx5`占用了，所以只能选择系统加速方式。

在Firefox上代理时注意要手动导入证书。Git这个工具则需要手动设置代理，对于代理端口的配置需要在Mate的控制中心→网络代理中查看。另外还需要配置`http.sslCAInfo`这个值来配置代理证书文件的路径（不建议用网上禁用SSL证书验证的方法）。

```sh
git config --global http.proxy 127.0.0.1:26561
git config --global http.sslCAInfo /home/xlbilly/.local/share/Steam++/SteamTools.Certificate.cer
```

## ~~Alacritty~~
一个不错的系统默认终端的替代品，使用这个的原因是发现它支持[Flexoki](https://stephango.com/flexoki)主题。~~由于Debian仓库中的版本较老，所以用yaml格式配置~~。~~目前已经换成Kitty终端代替了。~~

```yaml
# 老版本配置文件，已弃用
import:
  - ~/.config/alacritty/flexoki-dark.yaml # 加载主题
font:
  normal:
    family: Sarasa Term SC
    style: Regular
  bold:
    family: Sarasa Term SC
  italic:
    family: Sarasa Term SC
```

```toml
# ~/.config/alacritty/alacritty.toml
[font]
normal = { family = "等距更纱黑体 SC", style = "Regular" }
```

# Wezterm
目前WezTerm虽然不在官方仓库中，但有官方debian包。此终端自带flexoki主题。

使用lua文件配置：

```lua
-- ~/.config/wezterm/wezterm.lua

-- Pull in the wezterm API
local wezterm = require 'wezterm'

-- This will hold the configuration.
local config = wezterm.config_builder()

config.color_scheme = 'flexoki-dark'
config.font = wezterm.font '等距更纱黑体 SC'

-- Finally, return the configuration to wezterm:
return config
```

## ~~Kitty~~
~~**由于Kitty内存占用较大，而Alacritty的某些问题在换到fish之后不存在了，所以现在用Alacritty了**~~。目前换到了WezTerm。

在不使用alacrity后换的一个终端，目前用的最顺手的（主要原因是解决了alacrity和mate terminal的无法正确适配zsh-auto suggestions的问题）。

* 支持分屏
* 支持显示Shell Vim键位更改鼠标外形。

## ~~CopyQ~~
:::note
换到KDE后由于其已经内了剪贴板管理器，所以就不用这个了。在Windows上由于占用大，响应慢，所以暂时换成ditto了（虽然功能没前者多）。
:::

虽然Fcitx5按<kbd>Ctrl+;</kbd>键可以访问最近的剪贴板，但只能保存五条记录，所以装了一个剪贴板管理软件。

开源的剪贴板📋管理器，在Windows上（比自带的好很多）和Linux上体验都很不错。

## Vim
目前是当作一个多功能的轻量文本编辑器用，在Windows上也一样。配置就这些：

```vim
set guifont=Sarasa\ Term\ SC\ 12
set number
set nobackup
"GUI Options {{{1
set guioptions-=T
set guioptions-=e
set guioptions-=r
set guioptions-=l
set guioptions-=L
set guioptions-=t
set guioptions-=b
set guioptions-=m
set guioptions+=c
set guioptions+=!
set guioptions+=P
"}}}
set laststatus=2
set foldmethod=marker
set shiftwidth=4
if (has("win32") || has("win64"))
    set undodir=~/AppData/Local/Temp "防止相同目录下产生残余文件
endif
if has("gui")
    set background=dark
    colorscheme solarized
endif
```

[^1]: <https://bbs.archlinux.org/viewtopic.php?id=273679>

[^2]: <https://askubuntu.com/a/1051581>

[^3]: <https://www.instructables.com/Sync-Ipadiphone-With-LinuxDebianUbuntu/>
