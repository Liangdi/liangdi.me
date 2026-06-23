---
title: "记录一次 java 小项目的 rust 迁移过程"
date: 2022-04-26T11:06:35+08:00
draft: false
slug: "a-tiny-app-migrate-from-java-to-rust"
tags: ["rust","java","容器化"]
cover: "/p/a-tiny-app-migrate-from-java-to-rust/java-rust.png"
---

## 概述
使用 rust 也挺长时间了,但是一直是内部小打小闹,没有往客户那边推, 这次和客户商量好,拿一个很小的 java spring boot 项目开刀.

这个项目小到 tiny 级别, 主要功能是: 请求一个服务, 对数据进行业务逻辑处理后, 使用 freemarker 渲染呈现给终端用户, rest api 也就 10 多个.

## 迁移准备
首先,业务逻辑使用 rust 实现,肯定是没有大问题, 关键在于一些中间件和第三方库是否有替代,这样可以让运维或者交付更加平滑一点, 由于项目比较简单,第三方库也很通用,基本在 rust 这边也有对等的选择,所以本次迁移也将会简单很多.

## 迁移说明

### 库和中间件迁移
|库/中间件| Rust 版本 | Java 版本 | 备注 |
| ----| ----|----|----|
|Web Framework| actix-web 4| Spring Boot 2.6.x ||
|http cleint | reqwest | jdk 11 HttpClient ||
|json | serde_json | gson ||
|template | tera | freemarker ||
| logging | log4rs | spring boot logging ||
| assets embedded | rust-embed | spring boot static ||
| 配置 | dotenv | spring boot | 项目小, 配置需求不高, dotenv 够用|

### 有没有坑?
- 由于项目实在不复杂, 过程中也没有遇到特别的问题, rust 的库也能正常工作, 从代码的角度很多时候比 java 简洁, 当然集成没有 spring boot 那么简单, 比如配置的自动读取赋值, 模板需要手动集成等等.
- spring boot 的 rest request 配置, 对应在 actix-web 中也是很简单,使用 #[get], #[post] 等宏实现, 请求内容的解构也一样完善, 迁移过程中没有遇到什么问题.
- 模板方面, `tera` 是 `Jinja2/Django` like 的所以,使用起来也没有什么障碍.
- 战胜 rust 编译器后,程序运行,完成测试没有遇到什么 bug !

## 迁移后的变化
### 程序尺寸
- spring boot jar 大小 20MB, 即使是 spring-native 打包也要 60 多 mb, 并且 spring-native 打包的程序没法正常运行,因为 spring-native 还没有 GA ,所以也没有去深究.
- rust 的程序使用 `x86_64-unknown-linux-musl` target release 静态编译打包(默认配置,无其他优化), 程序 strip 后是 9.3MB , 相比 jar 也小了一半, 还没考虑 jdk.
- 容器化, 这点 rust 的优势就体现出来了, 使用 `FROM scratch` 打包出来的镜像大小是 9.75MB , 相比 spring boot 项目, 基本的 openjdk:17-alpine 就有 320 多 MB 加上项目程序,也要 340 多 MB 了.

### 性能
- 肉眼可见的启动块了, 模板渲染也快了
- 程序瓶颈在于远程的服务,所以当前程序性能测试并不一定可靠,所以并没有去做特别的测试
- 后续迁移更多项目后,再进行一次性能测试比较
## 思考
- 如果项目不复杂, java 迁移到 rust 还是不算复杂的.
- 容器化的收益是不小的,不管是启动速度还是镜像大小.
- 后面考虑将微服务化的 java 项目选择合适的项目再进行 rust 迁移.