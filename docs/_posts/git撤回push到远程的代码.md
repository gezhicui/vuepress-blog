---
title: git撤回push到远程的代码
date: 2021-03-16 20:50:15
tags:
  - git
categories:
  - git
permalink: /pages/7bec00/
sidebar: auto
author:
  name: 杨雨翔
  link: https://github.com/gezhicui
---

失误操作把错误的或者不完整的代码 commit 并且 push 到了远程。

可能会影响到远程上代码的正确性。不过好在 git 提供了撤回远程代码的方法。

## 具体操作过程

```
$ git log
-----------------------a-----------------------
commit 8622aca4a579bbb65c7255ae797622b4c33187a7 (HEAD -> master, origin/master, origin/HEAD)
Author: xxxcxy <yy_z3em@163.com>
Date:   Wed Apr 15 13:51:08 2020 +0800
-----------------------------------------------
    update.sh

------------------------b-----------------------
commit bc07480025bca168e2136064d795f2bb56eab999
Author: xxxcxy <yy_z3em@163.com>
Date:   Fri Apr 10 14:09:47 2020 +0800
------------------------------------------------
    add

commit 8bd321cd239abc9ebaf70810c7a094b9dec9dc63
Author: xxxcxy <yy_z3em@163.com>
Date:   Thu Apr 9 11:40:27 2020 +0800

    add

commit a0cd8a40263cd012c1ef2a80ef09ed31d9c37f42
Author: xxxcxy <yy_z3em@163.com>
Date:   Thu Apr 9 11:39:26 2020 +0800
```

a 部分的是刚刚 push 到远程的记录。

现在需要回滚到 b 部分的版本。

执行命令

> git reset --soft bc07480025bca168e2136064d795f2bb56eab999

查看 log

```
$ git log
commit bc07480025bca168e2136064d795f2bb56eab999 (HEAD -> master)
Author: xxxcxy <yy_z3em@163.com>
Date:   Fri Apr 10 14:09:47 2020 +0800

    add

commit 8bd321cd239abc9ebaf70810c7a094b9dec9dc63
Author: xxxcxy <yy_z3em@163.com>
Date:   Thu Apr 9 11:40:27 2020 +0800

    add

commit a0cd8a40263cd012c1ef2a80ef09ed31d9c37f42
Author: xxxcxy <yy_z3em@163.com>
Date:   Thu Apr 9 11:39:26 2020 +0800

    add LICENSE.
```

最上面 a 部分的 8622aca4a579bbb65c7255ae797622b4c33187a7 已经查不到了，这表示撤销成功了。

这个时候将本地的代码强制 push 到远程。

> git push origin master --force

撤回 push 到远程代码结束。
