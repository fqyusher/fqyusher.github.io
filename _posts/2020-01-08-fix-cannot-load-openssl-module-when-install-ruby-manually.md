---
layout: post
title:  "Solution:cannot load such file -- openssl (LoadError)"
date:   2020-01-08 20:30:11 +0800
category: [技能, 修复]
tags: [ruby，linux]
excerpt: "解决linux环境下手动安装ruby时报:cannot load such file -- openssl (LoadError)的错误"
---

> 问题

在linux环境下，手动安装ruby时，会报如下错误：

```text
cannot load such file -- openssl (LoadError)
```

> 解决方案

```sh
cd /ruby-dir/ext/openssl #进入ruby的安装包目录
ruby extconf.rb
make && make install
```

