---
title: "选择 podman 的理由, 以及它和 Kubernetes , Docker 的区别"
date: 2022-04-27T16:30:35+08:00
draft: false
slug: 'what-is-podman-and-different-from-kubernetes-and-docker'
tags: ["podman","docker","k8s","kubernetes"]
cover: "/p/what-is-podman-and-different-from-kubernetes-and-docker/podman-vs-docker.jpg"
---

## 前言
大家好,我是 Liangdi, podman 4.x 版本已经发布了, 我也从 docker 开始向 podman 迁移, 所以是合适的时候写点 podman 的文章了.

## podman 是什么
官方网站: [podman.io](https://podman.io)

官方自己的介绍: https://podman.io/whatis.html

名称 `podman` ,官方说明是 `Pod Manager` , 所以它不仅可以管理 OCI 容器,还可以管理 pod , 这也是和 docker 的最大差别吧. 

## 和 kubernetes 的区别

kubernetes(k8s) 是目前最流行的容器编排工具, 集群管理工具, 生态很完善, 也很`"重"`, pod 的概念就来自 k8s , 虽然 podman 也是管理 pod, 但是远远不及 k8s 的编排功能, 同时 podman 也没有集群管理功能,如果需要管理集群, 需要第三方工具完成.

所以 podman 定位也不是编排和集群管理工具, 紧紧是一个 pod 和容器的管理工具. 所以不是一个级别的东西, 这里不做太多的比较.

## 和 docker 的区别
如果仅仅从 `docker` 和 `podman` 两个命令提供的功能来讲,它们功能交集很大, podman 官方甚至推荐 `alias docker=podman` 来过渡.

- docker 文档更齐全,  podman 可以借用一下 docker 的文档
- docker 生态更加完善, podman 一时半会赶不上,但是如果你只是去跑容器, 那这是一样的
- docker 有 docker-compose, podman 早期没有对应工具,后面也出了 podman-compose, 但是这个功能是否必须? 值得考虑,因为 podman 支持 pod 管理.
- docker 有 machine , 让 windows 和 mac 支持 linux 容器, podman 也支持,而且已经比较完善.
- docker 有 docker-desktop , podman-desktop 目前还比较简单.
- docker 支持 rest api , podman 也支持 rest api, 这使得开发生态工具会比较简单.
- docker 有 swarm 支持集群部署, podman 没有对应工具, 不过支持 remote , 调用远程机器上的 podman service 执行对应的功能 , 这样能满足很多轻量化的场景.
- k8s 之前支持使用 docker-shim 和 docker 集成, 不过新版本也放弃这一层, 直接通过 CRI 调用 contained , podman 也不支持 CRI. 并且也没有什么计划.
- docker 商业/开源并行, podman 只有开源版本, 目前没有哪家公司提供商业支持(不清楚 redhat 有没有对应的服务,可能集成在订阅里面了).

## 为什么选 podman
上面讲了不少 docker 比 podman 有优势的方面, 这里开始讲 podman 的另外的东西, 这也是我选择 podman 主要原因.

先罗列一下 podman 适合的场景
- 没有很强的集群管理需求(或者说,已经有 overlay network 方案, podman 也是适用的)
- 仅仅为了容器化一些应用
- 团队内部轻量级使用,比如 ci/cd , 开发,测试环境等.
- 喜欢命令行或者脚本运维
- 感兴趣 podman 的生态建设(坑)

那么 podman 比 docker 好的方面有哪些呢?
- 更加 rootless , 尽管 docker 也可以 rootless, 但是 podman 设计之初就开始支持
- 没有 daemon , 这使得 podman 在结合 namespace 和 cgroup 一起使用会更加灵活
- pod , 和 k8s 基本一样的 pod , 一样支持 infra 容器. 这使得一些简单的容器`编排` 工作, podman 也可以简单实现.
- systemd service 集成, 由于没有 deamon , podman 通过 generate 子命令, 可以生成 systemd service 配置, 来管理容器和 pod 的作为服务启动. 
- k8s 关联, podman 可以生成 kubectl 的 yaml 配置文件, 也通过 podman play kube 来运行 k8s 的配置, 也可以作为 k8s 的一个过渡吧, 而且 podman 也没有去实现 CRI 的计划, 这应该也是官方的态度, 不会参和到 k8s 生态中, 保持自己的轻量化工具的定位吧.
- remote , podman 通过 ssh 隧道或者 tcp 端口, 可以连接到远程机器上的 podman service, 从而实现远程机器上的容器和 pod 管理.

所以 podman 提供了一些轻量化而又灵活的功能特性,满足容器化,以及小批量服务器的场景.
## 最后聊聊 podman 所在的 `https://github.com/containers` 组织

查看这个组织的仓库可以看到,他们真的是在做容器工具, 而且写了很多轮子, 包括 buildah 和 skopeo 这两个工具,与 podman 一起被称为下一代容器工具.

runc 是一个有名的 low-level OCI rumtime, 他们就开发了一个 crun . podman 早期版本使用 runc, 最新的版本已经使用 crun 了.

众所周知, golang 是容器生态的主要语言, podman 也是 go 写的,但是在 podman 4 的版本中, podman 增加了非 CNI 的网络栈支持, 这几个工具是 netavark 和 aardvark-dns, 这两个工具是 rust 写的, 而且还有 youki 这个 rust 写的 low-level OCI runtime, 不知道将来某一天 podman 会不会默认使用 youki , 还有好几个 rust 写的容器技术相关的应用和库, 这是要与 golang 分天下的节奏.

如果你也对 podman 以及其生态感兴趣, 关注我吧, 我会给你带来 podman 最新的动态以及实践方案.

我的技术 Blog:[https://liangdi.me](https://liangdi.me)