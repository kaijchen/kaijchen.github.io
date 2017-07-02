---
layout: post
title:  "Debian更改gdm3主题"
date:   2017-05-26 12:09:23 +0800
categories: notes
---

[debian wiki: GDM](https://wiki.debian.org/GDM)

> 1. Edit /etc/gdm3/greeter.dconf-defaults as root
> 2. Uncomment and/or modify the desired settings
> 3. Save and close the editor
> 4. Finally, run as root: dpkg-reconfigure gdm3

片段示例，其中icon-theme那行需手动添加

```
/etc/gdm3/greeter.dconf-defaults
---

#  - Change the GTK+ theme
[org/gnome/desktop/interface]
gtk-theme='Arc-Darker'
icon-theme='Arc'
cursor-theme='Breeze'

```
