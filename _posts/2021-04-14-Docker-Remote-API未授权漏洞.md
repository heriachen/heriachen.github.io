---
layout:     post
title:      Docker Remote API未授权漏洞
subtitle:   Docker Remote API未授权
date:       2021-04-14
author:     heria
header-img: img/post-bg-008.jpeg
catalog: true
tags:
---

## Docker Remote API未授权漏洞

### 漏洞描述

Docker 是一个开源的应用容器引擎，允许开发者将其应用和依赖包打包到一个可移植的容器中，并发布到任何流行的 Linux 机器上，以实现虚拟化。

Docker 的 Remote API 因配置不当可以未经授权进行访问，从而被攻击者恶意利用。

攻击者无需认证即可访问到 Docker 数据，可能导致敏感信息泄露，黑客也可以恶意删除 Docker 上的数据；攻击者可进一步利用 Docker 自身特性，直接访问宿主机上的敏感信息，或对敏感文件进行修改，最终完全控制服务器。

### 影响范围

对公网开放，且未启用认证的 Docker Remote API 接口。

### 漏洞利用

内网检测过程中通过nmap或goby可发现docker未授权问题

![image-20210414100030451](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210414100030451.png)

访问ip/version

![image-20210414100123468](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210414100123468.png)

攻击机需安装docker

```
docker -H <host>:<port> info
```

可查看到一些信息

![image-20210414100347472](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210414100347472.png)

![image-20210414100434750](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210414100434750.png)

查看运行容器信息

```
docker -H <host>:<port> ps -a
```

![image-20210414100640352](https://raw.githubusercontent.com/heriachen/cloudimg/main/img2/image-20210414100640352.png)



也可进入容器

```
docker -H <host>:<port> exec -it <container name> /bin/bash
```



网上看到进一步利用的帖子，可参考

> https://zgao.top/docker-remote-api%E6%9C%AA%E6%8E%88%E6%9D%83%E8%AE%BF%E9%97%AE%E6%BC%8F%E6%B4%9E%E5%A4%8D%E7%8E%B0/
>

### 影响范围

### 修复方案

- **修改 Docker Remote API 服务默认参数**。

  **注意**：该操作需要重启 Docker 服务才能生效。

  修改 Docker 的启动参数：

  - 定位到 DOCKER_OPTS 中的 `tcp://0.0.0.0:2375`，将`0.0.0.0`修改为`127.0.0.1`

- **为 Remote API 设置认证措施**。参照 https://docs.docker.com/engine/api/配置 Remote API 的认证措施。

  **注意**：该操作需要重启 Docker 服务才能生效。

- **修改 Docker 服务运行账号**。请以较低权限账号运行 Docker 服务；另外，可以限制攻击者执行高危命令。

  **注意**：该操作需要重启 Docker 服务才能生效。

- **设置防火墙策略**。如果正常业务中 API 服务需要被其他服务器来访问，可以配置安全组策略或 iptables 策略，仅允许指定的 IP 来访问 Docker 接口。

