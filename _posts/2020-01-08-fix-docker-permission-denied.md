---
layout: post
title:  "docker用户权限解决方案"
date:   2020-01-08 19:22:11 +0800
category: [技能, 修复]
tags: [docker]
excerpt: "docker用户权限解决方案"
---

> 问题

非root用户执行docker相关命令时，会出现如下错误：

```text
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.39/containers/json: dial unix /var/run/docker.sock: connect: permission denied
```

> 原因

docker进程使用Unix Socket而不是使用TCP端口，而默认情况下，Unix Socket属于root用户，所以非root用户不能访问。

> 解决方案

- 使用`sudo`命令

  ```sh
  sudo docker ps -a
  ```

  缺点：每次带`sudo`不方便，还得输密码。

- 将用户加入`docker`用户组

  ```sh
  sudo gpasswd -a $USER docker #将当前用户加入docker组
  newgrp docker #更新用户组
  docker ps -a #docker命令
  ```

  原因：安装docker时，会创建docker用户组，会默认赋予Unix Socket的权限。