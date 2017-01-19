---
layout: post
title:  "用find找出一段时间内的文件"
date:   2017-01-19 16:38:10 +0800
categories: notes
---

### 找出 2006 年 1 月 1 日 ~ 1 月 8 日的文件

```shell
$ touch --date "2006-01-01" /tmp/start
$ touch --date "2006-01-09" /tmp/end
$ find /path -type f -newer /tmp/start -not -newer /tmp/end
```

### 找出 2006 年 6 月份的文件

```shell
$ find /path -type f -exec ls -l --time-style=long-iso {} \; | grep "2006-06"
```

### 找出 10 天前、 90 天内改动的文件

```shell
$ find /path -type f -mtime -90 -mtime +10
```

### 找出 5 分钟前, 半个小时内改动的文件

```shell
$ find /path -type f -mmin -30 -mmin +5
```

## 文件改动标识

```
atime 最后一次访问时间, 如 ls, more 等,
      但 chmod, chown, ls, stat 等不会修改些时间,
      使用 ls -utl 可以按此时间顺序查看;

ctime 最后一次状态修改时间, 如 chmod, chown 等,
      状态时间改变但修改时间不会改变, 使用 stat file 可以查看;

mtime 最后一次内容修改时间, 如 vi 保存后等,
      修改时间发生改变的话, atime 和 ctime 也相应跟着发生改变.
```

