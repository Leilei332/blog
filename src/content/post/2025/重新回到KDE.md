---
title: 重新回到KDE
description: 无描述
publishDate: 2025-01-04
updatedDate: "2025-03-15"
tags: 
- Linux
- Blog

slug: return-to-kde
---


我之前在[](<#Debian 12重新体验>)提到过不用KDE的原因是Discover过于卡顿，现在决定用回KDE了。在Debian上用`tasksel`就可以很方便地安装KDE桌面。

:::note
由于上一篇文章还没有写完，所以此文章仅记录与KDE配置有关的内容。
:::

Discover的正常运行依赖`packagekit`，把这个服务禁用了Discover占用就会变小：

```sh
sudo systemctl disable packagekit.service
```

# 软件
## QAPT安装器
之前在MATE上用的[GDebi](https://packages.debian.org/bookworm/gdebi)是基于Python的，导致占用非常大。目前正在使用的`qapt-deb-installer`虽然启动有点慢，但是解决了占用过大的问题。

## 更多设置界面
推荐安装这两个包：

1. `kde-config-plymonth` 配置
2. `kde-config-fcitx5`

# 调整
KDE中仍然会有一些动画导致运行很卡，目前的优化是：

* 快速设置页面将“动效速度”拉至最右
* 工作区行为->桌面特效，关闭没用的特效

## Kimpanel
推荐使用KDE的原生的“输入法面板”代替默认的托盘图标，这样的输入法可以将功能按钮显示在任务栏上，非常好看（并且有*Papirus*主题适配）。

## 禁用Akonadi
**Aknoadi这个功能可以说是KDE很慢的元凶之一**，因为它会启动一个占用很多的`mysqld`服务。编辑`~/.config/akonadi/akonadiserverrc`然后改这项：

```ini
[QMYSQL]
StartServer=false
```