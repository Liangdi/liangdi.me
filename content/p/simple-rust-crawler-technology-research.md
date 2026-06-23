---
title: "一次简单的 rust 爬虫开发技术调研"
date: 2022-05-06T00:38:25+08:00
draft: false
slug: "simple-rust-crawler-technology-research"
tags: ["rust","爬虫","WebDriver"]
cover: "/p/simple-rust-crawler-technology-research/rust-crawler.png"
---

## 前言
看到标题,是不是觉得我疯了,爬虫,这个时间点应该还轮不到 rust 吧!

确实,现在成熟的爬虫技术基本使用 python, java 等,那么这篇文章的用意是什么呢? 我先来交代一下背景, 2017 年,我挖了一个坑:
[https://zhuanlan.zhihu.com/p/24900305](https://zhuanlan.zhihu.com/p/24900305)

这个项目后来商用了,所以就不方便继续更新, 到了现在 2022 年了, 各种技术已经迭代了好多次, 那么,我也要重新思考一下一套新的爬虫架构了,这套架构基于云原生,容器化,结合不同语言取长补短,相互配合,简单概括如下:
- 爬虫节点容器运行,语言无关,所以会支持 python ,java 等各语言开发的爬虫
- 统一的 DAG 任务分发调度节点
- 统一的数据集接口

其中一点是爬虫语言无关,所以会设计统一的输入输出接口,让各语言编写的程序都能在容器中执行, python, java, golang 的爬虫生态都很不错了,文章也很多, rust 这方面比较少,所以这里就写一篇 rust 的文章.

## 爬虫底层关注的技术点
- http 库
- html 解析
- json 解析
- WebDriver 集成

## http 库
作为爬虫的 http 库, 只要支持 http 协议基本就 ok, 主要是设置 header, 设置 cookies, 设置代理等等.

rust 的 http 库列表 [https://lib.rs/web-programming/http-client](https://lib.rs/web-programming/http-client)

这里使用 reqwest 做个简单的示例
```rust
use reqwest;
use http::{HeaderMap, HeaderValue, header::{COOKIE,USER_AGENT}};
#[tokio::main]
async fn main()  -> Result<(),reqwest::Error> {

    // HTML
    let url = "https://ssr1.scrape.center/";
    let resp = reqwest::get(url).await?;
    println!("Response: {:?} {}", resp.version(), resp.status());
    println!("Headers: {:#?}\n", resp.headers());
    //println!("Body:{:#?}",resp.text().await?);

    // JSON
    let url = "https://spa1.scrape.center/api/movie/?limit=10&offset=0";
    let resp = reqwest::get(url).await?;
    let json_body:serde_json::Value = resp.json().await?;
    println!("Json:{:#?}",json_body);

    // POST
    let _resp = reqwest::Client::new()
        .post(url)
        .form(&[("name", "value")])
        .send()
        .await;
    // Header
    let _resp = reqwest::Client::new()
        .post(url)
        .header("Auth","xxx")
        
        .send()
        .await;
        
    // UserAgent Cookies
    let client = reqwest::Client::builder()
        .cookie_store(true).build().unwrap();
    let mut headers = HeaderMap::new();
    headers.insert(COOKIE, HeaderValue::from_str("key=value").unwrap());
    headers.insert(USER_AGENT,HeaderValue::from_str("my user-agent").unwrap());
    let _reps = client.get(url)
        .headers(headers)
        .send()
        .await?;
    Ok(())
}

```
Cargo.toml
```bash
[dependencies]
http = "0.2.7"
reqwest = {version = "0.11.10",features = ["json","cookies"] }
serde_json = "1.0.81"
tokio = {version ="1.18.2",features = ["full"] }

```

## html/json 解析
采集到 html 和 json 数据后,需要解析出有用的数据, html 内容主要用 css 选择器筛选, json 可以转 map 或者使用 jsonpath 筛选, 这里使用 scraper 和 jsonpath 这两个库做个简单示例.

Cargo
```bash
cargo add scraper
cargo add jsonpath
```

```rust
use reqwest;
use scraper::{Html, Selector};
#[tokio::main]
async fn main()  -> Result<(),reqwest::Error> {

    // HTML
    let url = "https://ssr1.scrape.center/";
    let resp = reqwest::get(url).await?;
    //println!("Body:{:#?}",resp.text().await?);
    let body = resp.text().await?;
    let doc = Html::parse_fragment(&body);
    let selector = Selector::parse(".m-b-sm").unwrap();
    for el in doc.select(&selector) {
        println!("title:{}",el.inner_html());
    }
    // JSON
    let url = "https://spa1.scrape.center/api/movie/?limit=10&offset=0";
    let resp = reqwest::get(url).await?;
    let json_body:serde_json::Value = resp.json().await?;
    //println!("Json:{:#?}",json_body);
    let json_sel = jsonpath::Selector::new("$.results.*.name").unwrap();
    for json_el in json_sel.find(&json_body) {
        println!("json title:{}",json_el.as_str().unwrap());
    }
    
    Ok(())
}

```
运行结果
```bash
title:霸王别姬 - Farewell My Concubine
title:这个杀手不太冷 - Léon
title:肖申克的救赎 - The Shawshank Redemption
title:泰坦尼克号 - Titanic
title:罗马假日 - Roman Holiday
title:唐伯虎点秋香 - Flirting Scholar
title:乱世佳人 - Gone with the Wind
title:喜剧之王 - The King of Comedy
title:楚门的世界 - The Truman Show
title:狮子王 - The Lion King
json title:霸王别姬
json title:这个杀手不太冷
json title:肖申克的救赎
json title:泰坦尼克号
json title:罗马假日
json title:唐伯虎点秋香
json title:乱世佳人
json title:喜剧之王
json title:楚门的世界
json title:狮子王
```

## WebDriver 集成
一些网站由于反爬,构建 http 请求会遇到一些额外问题,那么使用 webdriver 将会是另外一种爬虫解决方案, 这里使用 `thirtyfour` 这个库

Cargo
```bash
cargo add thirtyfour
cargo add tokio
```
先运行 `webdriver`, 我这里使用 chromedriver , 默认监听 9515 端口

示例代码
```rust

use thirtyfour::{prelude::*, error::WebDriverError};
use tokio;

#[tokio::main]
async fn main() -> Result<(), WebDriverError>{
    let url = "https://spa1.scrape.center/";
    let caps = DesiredCapabilities::chrome();
    
    let driver = WebDriver::new("http://localhost:9515", caps).await?;
    driver.get(url).await?;
    // 等待我们要的元素
    let check = driver.query(By::Css(".m-b-sm")).first().await?;
    check.wait_until().displayed().await?;
    let els = driver.find_elements(By::Css(".m-b-sm")).await?;
    for el in els {
        println!("el:{}",el.inner_html().await?.as_str());
    }
    driver.quit().await?;
    Ok(())
}

```
运行结果
```bash
el:霸王别姬 - Farewell My Concubine
el:这个杀手不太冷 - Léon
el:肖申克的救赎 - The Shawshank Redemption
el:泰坦尼克号 - Titanic
el:罗马假日 - Roman Holiday
el:唐伯虎点秋香 - Flirting Scholar
el:乱世佳人 - Gone with the Wind
el:喜剧之王 - The King of Comedy
el:楚门的世界 - The Truman Show
el:狮子王 - The Lion King
```

## 简单总结
- 支持爬虫操作的基础库基本都是涵盖的
- 相关的示例代码或者教程还是缺少
- 这次验证下来,还是可以满足我新系统设计的需求的,可以使用 rust 开发爬虫节点



