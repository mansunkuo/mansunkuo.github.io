---
title: 'Sling'
date: 2024-05-12T23:12:18+08:00
# weight: 1
# aliases: ["/first"]
tags:
  - Data Engineering
  - Extract & Load
author: "Mansun Kuo"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
# description: "Desc Text."
# canonicalURL: "https://canonical.url/to/page"
disableHLJS: false # to disable highlightjs
disableShare: true
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
---

最近在工作上剛好有異質資料庫搬遷的需求，以往在處理類似的需求的時候，往往需要自行處理一些型別轉換和資料讀出寫入的邏輯，大部分的工程師，尤其是資料工程師，多少也可能自己處理過類似的邏輯，若需要更長一段時間來驗證新舊資料庫的差異，本質上其實也就是一般在資料工程最常處理的起手式－擷取與載入 (Extract & Load)。這篇文章會簡單介紹 ELT (Extract, load, and transform) 的基本概念，並帶大家認識一下 Sling 的一些基本功能。

## ELT 與 ETL

ELT 相較於歷史較為悠久的 ETL (Extract, transform, and load)，都是在進行進一步資料分析前的處理程序，兩者的第一步都是先從來源初將資料擷取出來，其中 ETL 會在載入目標資料庫前先藉由額外的的運算資源來處理資料轉換的邏輯，而 ELT 則著重於讓資料先落地在目標資料庫，所需的轉換可以之後再根據實際的分析需進行轉換，藉由近代資料倉儲強大的運算能力讓資料處理的過程更有彈性。近期也看到一些相關的框架漸漸往更輕量且更容易整合和使用的方向靠攏，例如 Dagster 就提出了所謂 Embedded ELT 的概念，藉由整合 pipeline engine 和輕量的框架，讓資料落地這件事可以用更簡單的方式完成。

## 什麼是 Sling

[Sling](https://slingdata.io/) 是一款強大的數據整合 CLI 工具，其主要負責的任務在於簡化不同資料庫和檔案之間 EL (Extract & Load) 的任務，從他們的 Connectors 可以看到 Sling 主要負責四種類型的資料搬遷：
- 資料庫到資料庫
- 檔案到資料庫
- 資料庫到檔案
- 檔案到檔案

聽起來感覺包山包海




## 參考文獻
- [Sling](https://slingdata.io/)
- [Stop Reinventing Orchestration: Embedded ELT in the Orchestrator](https://dagster.io/blog/dagster-embedded-elt)
- [Sling Out Your ETL Provider with Embedded ELT](https://dagster.io/blog/sling-out-your-etl-provider-with-embedded-elt)
- [What’s the Difference Between ETL and ELT?](https://aws.amazon.com/compare/the-difference-between-etl-and-elt/)