---
layout: post
title:  "vim换行符转换"
date:   2016-10-09 11:27:45 +0800
categories: notes
---
`unix`、`dos`(windows)、`mac`(旧mac os，OS X 之前) 使用的换行符格式是不一样的，  
所以传文件时需要修改换行符格式

    unix    LF only (each line ends with an LF character).
            //Unix based systems and Mac OS X and later.
    
    dos     CRLF (each line ends with CR then LF).
            //DOS and Windows.
    
    mac     CR only (each line ends with a CR character).
            //Mac OS version 9 and earlier

#### 在vim下转换换行符格式

只需要以下命令，分别设置成 `unix` `dos` `mac`

    :set ff=unix
    
    :set ff=dos
    
    :set ff=mac

#### 转换换行符格式混合的文件

（vim 7.2.40 及更新版本）

`dos/unix` 转换成 `unix`

    :update             Save any changes.
    :e ++ff=dos         Edit file again, using dos file format.
    :setlocal ff=unix   This buffer will use LF-only line endings when written.
    :w                  Write buffer using unix (LF-only) line endings.

`unix/dos` 转换成 `dos`

    :update         Save any changes.
    :e ++ff=dos     Edit file again, using dos file format.
    :w              Write buffer using dos (CRLF) line endings.

#### 更换默认换行符

    :set ffs=unix,dos
    //读取文件时，侦测unix,dos换行符格式（顺序无关）
    //创建文件时默认第一项为换行符格式


参考：[http://vim.wikia.com/wiki/File_format](http://vim.wikia.com/wiki/File_format)
