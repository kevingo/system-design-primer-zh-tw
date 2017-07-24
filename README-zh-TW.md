*[English](README.md) ∙ [简体中文](README-zh-Hans.md) | [Brazilian Portuguese](https://github.com/donnemartin/system-design-primer/issues/40) ∙ [Turkish](https://github.com/donnemartin/system-design-primer/issues/39) | [Add Translation](https://github.com/donnemartin/system-design-primer/issues/28)*

# 系統設計入門

<p align="center">
  <img src="http://i.imgur.com/jj3A5N8.png">
  <br/>
</p>

## 動機

> 學習如何設計大型系統。
>
> 準備系統設計的面試。

### 學習如何設計大型系統

學習如何設計可擴展的系統會幫助你成為一個更好的工程師。

系統設計是一個廣泛的主題。在網路上，關於**系統設計的資源也是不及其數**。

本專案是一個 **組織好的資源集合**，幫助你學習如何建構可擴展的系統。

### 從開放原始碼社群中學習

這是一個早期、持續不斷更新的開放原始碼專案。

[任何貢獻](#contributing) 都相當歡迎！

### 準備系統設計的面試

除了程式的面試外，系統設計在許多科技公司的**面試流程**中都是**必要**的一環。

**練習常見的系統設計問題**，並且將你的設計結果和**參考答案**進行**比對**：討論、程式碼和圖表。

關於面試的其他主題：

* [學習指南](#study-guide)
* [如何處理系統設計面試問題](#how-to-approach-a-system-design-interview-question)
* [系統設計面試問題與**解答**](#system-design-interview-questions-with-solutions)
* [物件導向設計問題與**解答**](#object-oriented-design-interview-questions-with-solutions)
* [其他的系統設計面試問題](#additional-system-design-interview-questions)

## 學習單字卡

<p align="center">
  <img src="http://i.imgur.com/zdCAkB3.png">
  <br/>
</p>

底下提供的[學習單字卡](https://apps.ankiweb.net/)以每隔一段時間間隔出現的方式幫助你學習系統設計的概念。

* [系統設計單字卡](resources/flash_cards/System%20Design.apkg)
* [系統設計練習單字卡](resources/flash_cards/System%20Design%20Exercises.apkg)
* [物件導向設計練習單字卡](resources/flash_cards/OO%20Design.apkg)

這些是非常棒的學習資源，隨時都可以使用。

### 程式設計學習資源：互動式程式學習設計

你正在尋找資源來面對 [**程式語言面試**](https://github.com/donnemartin/interactive-coding-challenges)嗎？

<p align="center">
  <img src="http://i.imgur.com/b4YtAEN.png">
  <br/>
</p>

請參考 [**互動程式語言學習挑戰**](https://github.com/donnemartin/interactive-coding-challenges)，當中還包含了底下的學習單字卡：

* [程式語言學習單卡](https://github.com/donnemartin/interactive-coding-challenges/tree/master/anki_cards/Coding.apkg)

## 如何貢獻

> 從社群中學習

隨時歡迎提交 Pull Request：

* 修正錯誤
* 改善章節內容
* 增加新的章節
* [翻譯](https://github.com/donnemartin/system-design-primer/issues/28)

某些還需要再完善的章節放在 [修正中](#under-development) 中。

請參考 [貢獻指南](CONTRIBUTING.md)。

## 系統設計主題的索引

> 底下是數個系統設計的主題， 包含了優點及缺點。**要記得，每一個系統設計的考量都包含著某種取捨**。
>
> 每一章節都包含更深入的資源的連結。

<p align="center">
  <img src="http://i.imgur.com/jrUBAF7.png">
  <br/>
</p>

* [系統設計主題：從這裡開始](#system-design-topics-start-here)
    * [第一步： 複習關於可擴展性的影片講座](#step-1-review-the-scalability-video-lecture)
    * [第二步： 複習關於可擴展性的文章](#step-2-review-the-scalability-article)
    * [下一步](#next-steps)
* [效能與可擴展性](#performance-vs-scalability)
* [延遲與吞吐量](#latency-vs-throughput)
* [可用性與一致性](#availability-vs-consistency)
    * [CAP 理論](#cap-theorem)
        * [CP - 一致性與分區容錯性](#cp---consistency-and-partition-tolerance)
        * [AP - 可用性與分區容錯性](#ap---availability-and-partition-tolerance)
* [一致性模式](#consistency-patterns)
    * [弱一致性](#weak-consistency)
    * [最終一致性](#eventual-consistency)
    * [強一致性](#strong-consistency)
* [可用模式](#availability-patterns)
    * [錯誤轉移](#fail-over)
    * [複寫機制](#replication)
* [域名系統](#domain-name-system)
* [內容傳遞網路(CDN)](#content-delivery-network)
    * [CDN 推送](#push-cdns)
    * [CDN 拉取](#pull-cdns)
* [負載平衡器](#load-balancer)
    * [主動到備用切換模式(AP Mode)](#active-passive)
    * [雙主動切換模式(AA Mode)](#active-active)
    * [四層負載平衡](#layer-4-load-balancing)
    * [七層負載平衡](#layer-7-load-balancing)
    * [水平擴展](#horizontal-scaling)
* [反向代理 (網路伺服器)](#reverse-proxy-web-server)
    * [負載平衡和反向代理](#load-balancer-vs-reverse-proxy)
* [應用層](#application-layer)
    * [微服務](#microservices)
    * [服務發現](#service-discovery)
* [資料庫](#database)
    * [關聯式資料庫管理系統 (RDBMS)](#relational-database-management-system-rdbms)
        * [Master-slave 複製](#master-slave-replication)
        * [Master-master 複製](#master-master-replication)
        * [聯合](#federation)
        * [分片](#sharding)
        * [反正規化](#denormalization)
        * [SQL 優化](#sql-tuning)
    * [NoSQL](#nosql)
        * [鍵值儲存](#key-value-store)
        * [文件儲存](#document-store)
        * [寬列儲存](#wide-column-store)
        * [圖形資料庫](#graph-database)
    * [SQL或NoSQL](#sql-or-nosql)
* [快取](#cache)
    * [客戶端快取](#client-caching)
    * [CDN 快取](#cdn-caching)
    * [網站伺服器快取](#web-server-caching)
    * [資料庫快取](#database-caching)
    * [應用程式快取](#application-caching)
    * [資料庫查詢級別的快取](#caching-at-the-database-query-level)
    * [物件級別的快取](#caching-at-the-object-level)
    * [如何更新快取](#when-to-update-the-cache)
        * [快取端](#cache-aside)
        * [直寫模式](#write-through)
        * [回寫模式](#write-behind-write-back)
        * [重新整理](#refresh-ahead)
* [異步](#asynchronism)
    * [消息隊列](#message-queues)
    * [工作隊列](#task-queues)
    * [反壓機制](#back-pressure)
* [通訊](#communication)
    * [傳輸控制通訊協定 (TCP)](#transmission-control-protocol-tcp)
    * [使用者封包通訊協定 (UDP)](#user-datagram-protocol-udp)
    * [遠端程序呼叫 (RPC)](#remote-procedure-call-rpc)
    * [REST](#representational-state-transfer-rest)
* [安全](#security)
* [附錄](#appendix)
    * [2 的次方表](#powers-of-two-table)
    * [每個開發者都應該知道的延遲數量](#latency-numbers-every-programmer-should-know)
    * [其他的系統設計面試問題](#additional-system-design-interview-questions)
    * [真實世界的架構](#real-world-architectures)
    * [公司的系統架構](#company-architectures)
    * [公司的工程部落格](#company-engineering-blogs)
* [尚在進行中](#under-development)
* [致謝](#credits)
* [聯絡方式](#contact-info)
* [授權](#license)

## 學習指南

> 基於你面試的時間 (短、中、長) 來複習這些建議的主題。

![Imgur](http://i.imgur.com/OfVllex.png)

**Q: 對於面試者來說，我需要知道這裡所有的知識嗎？**

**A: 不，如果是為了面試，你不需要知道這裡所有的知識**。

在一場面試中，你會被問到什麼問題取決於以下幾點：

* 你有多少經驗
* 你的技術背景
* 你面試什麼職位
* 你面試什麼公司
* 你的幸運程度

越有經驗的面試者通常被期望瞭解更多系統設計的知識。如果是架構師或團隊負責人則被預期瞭解更多除了個人貢獻以外的知識。頂尖的科技公司通常也會有一次或多次的系統設計面試。

面試時，會很廣泛地展開，並在幾個特定領域深入探討。這裡會幫助你了解一些系統設計不同的主題，基於你面試的時間、經驗、職位和公司來調整你所需要涉獵的知識內容。

* **短期** - 以系統設計的**廣度**為目標。通過解決**一些**面試題目來練習。
* **中期** - 以系統設計的**廣度**和**初級深度**為目標。通過解決**很多**面試題目來練習。
* **長期** - 以系統設計主題的**廣度**和**高級深度**為目標。通過解決**大部分**面試題目來練習。

| | 短期 | 中期 | 長期 |
|---|---|---|---|
| 閱讀 [系統設計主題](#index-of-system-design-topics) 來取得關於系統如何運作的廣泛知識 | :+1: | :+1: | :+1: |
| 閱讀一些你要面試的 [公司技術部落格](#company-engineering-blogs) 文章 | :+1: | :+1: | :+1: |
| 閱讀關於 [真實世界的架構](#real-world-architectures) | :+1: | :+1: | :+1: |
| 複習 [如何處理一個系統設計的面試題目](#how-to-approach-a-system-design-interview-question) | :+1: | :+1: | :+1: |
| 完成 [系統設計面試題目與解答](#system-design-interview-questions-with-solutions) | 一些 | 很多 | 大部分 |
| 完成 [物件導向設計與解答](#object-oriented-design-interview-questions-with-solutions) | 一些 | 很多 | 大部分 |
| 複習 [其他的系統設計面試提](#additional-system-design-interview-questions) | 一些 | 很多 | 大部分 |

## 如何解決一個系統設計的面試題目

> 如何面對一個系統設計的面試題目

系統設計是一個**開放式的對話過程**，面試官會期望由你來主導這個對話。

你可以使用下面的步驟來引導整個討論過程。為了鞏固這個流程，請使用下面的步驟完成[系統設計的面試題和解答](#system-design-interview-questions-with-solutions)這個章節。

### 第一步：描述使用的場景、限制及假設

把問題的需求和範圍等資訊搜集起來。詢問以下問題，讓我們可以明確的定義使用場景和限制，對於提出的假設進行討論：

* 誰會使用這個系統？
* 他們怎麼使用系統？
* 有多少使用者？
* 系統的作用是什麼？
* 系統的輸入和輸出是什麼？
* 我們預期希望處理多少資料？
* 我們希望每秒處理多少請求？
* 預期的讀、寫比例為何？

### 第二步：建立一個高維度的設計

使用重要的元件來建立一個高維度的設計。

* 畫出主要的元件與連接情況
* 證明你的想法

### 第三步： 設計核心的元件

對每一個核心元件進行深入的分析。舉例來說， 如果你被問到[設計一個短網址的服務](solutions/system_design/pastebin/README.md)，底下可以開始討論：

* 產生並儲存一個完整網址的 Hash
    * [MD5](solutions/system_design/pastebin/README.md) 和 [Base62](solutions/system_design/pastebin/README.md)
    * Hash 碰撞
    * SQL 或 NoSQL
    * 資料庫的模型
* 將一個 hash 過後的網址轉換為完整的網址
    * 資料庫查詢
* API 和物件導向設計

### 第四步：評估你的設計

確認及指出你的設計的瓶頸與限制，舉例來說，你需要底下幾個元件來擴展你的設計嗎？

* 負載平衡
* 水平擴展
* 快取
* 資料庫切片

針對你的設計討論可能的解決方法與代價。每個設計都有取捨。使用[可擴展的設計原則](#index-of-system-design-topics)來處理系統瓶頸。

### 快速有效的進行估算

你可能要求針對設計進行一些估算，可以參考[附錄](#appendix)的一些資源：

* [使用快速估算法](http://highscalability.com/blog/2011/1/26/google-pro-tip-use-back-of-the-envelope-calculations-to-choo.html)
* [2 的次方表](#powers-of-two-table)
* [每個開發者都應該知道的延遲數量](#latency-numbers-every-programmer-should-know)

### 相關資源與延伸閱讀

查看以下的連結獲得更好的做法：

* [如何在系統設計的面試中勝出](https://www.palantir.com/2011/10/how-to-rock-a-systems-design-interview/)
* [系統設計的面試](http://www.hiredintech.com/system-design)
* [系統架構與設計的面試介紹](https://www.youtube.com/watch?v=ZgdS0EUmn70)

## 系統設計面試問題與解答

> 常見的系統設計面試問題與相關範例的討論、程式碼以及圖表。
>
> 相關的解答位於 `解答/` 的資料夾中。

| 問題 | |
|---|---|
| 設計 Pastebin.com (或 Bit.ly) | [解答](solutions/system_design/pastebin/README.md) |
| 設計一個像是 Twitter 的 timeline (或 Facebook feed)<br/>設計一個 Twitter 搜尋功能 (or Facebook 搜尋功能) | [解答](solutions/system_design/twitter/README.md) |
| 設計一個爬蟲系統 | [解答](solutions/system_design/web_crawler/README.md) |
| 設計 Mint.com 網站 | [解答](solutions/system_design/mint/README.md) |
| 設計一個社交網站的資料結構 | [解答](solutions/system_design/social_graph/README.md) |
| 設計一個搜尋引擎使用的鍵值儲存資料結構 | [解答](solutions/system_design/query_cache/README.md) |
| 設計一個根據產品分類的亞馬遜銷售排名 | [解答](solutions/system_design/sales_rank/README.md) |
| 在 AWS 上設計一個百萬用戶等級的系統 | [解答](solutions/system_design/scaling_aws/README.md) |
| 增加一個系統設計的問題 | [貢獻](#contributing) |

### 設計 Pastebin.com (或 Bit.ly)

[閱讀練習與解答](solutions/system_design/pastebin/README.md)

![Imgur](http://i.imgur.com/4edXG0T.png)

### 設計一個像是 Twitter 的 timeline (或 Facebook feed)設計一個 Twitter 搜尋功能 (or Facebook 搜尋功能)

[閱讀練習與解答](solutions/system_design/twitter/README.md)

![Imgur](http://i.imgur.com/jrUBAF7.png)

### 設計一個爬蟲系統

[閱讀練習與解答](solutions/system_design/web_crawler/README.md)

![Imgur](http://i.imgur.com/bWxPtQA.png)

### 設計 Mint.com 網站

[閱讀練習與解答](solutions/system_design/mint/README.md)

![Imgur](http://i.imgur.com/V5q57vU.png)

### 設計一個社交網站的資料結構

[閱讀練習與解答](solutions/system_design/social_graph/README.md)

![Imgur](http://i.imgur.com/cdCv5g7.png)

### 設計一個搜尋引擎使用的鍵值儲存資料結構 

[閱讀練習與解答](solutions/system_design/query_cache/README.md)

![Imgur](http://i.imgur.com/4j99mhe.png)

### 設計一個根據產品分類的亞馬遜銷售排名

[閱讀練習與解答](solutions/system_design/sales_rank/README.md)

![Imgur](http://i.imgur.com/MzExP06.png)

### 在 AWS 上設計一個百萬用戶等級的系統

[閱讀練習與解答](solutions/system_design/scaling_aws/README.md)

![Imgur](http://i.imgur.com/jj3A5N8.png)

## 物件導向設計面試問題與解答

> 常見的物件導向面試問題與案例探討、程式碼與圖表。
>
> 相關的答案為在 `解答/` 目錄中。

>**注意: 本章節仍在完善內容中**

| 問題 | |
|---|---|
| 設計一個 hash map | [Solution](solutions/object_oriented_design/hash_table/hash_map.ipynb)  |
| 設計一個 LRU 快取 | [Solution](solutions/object_oriented_design/lru_cache/lru_cache.ipynb)  |
| 設計一個客服系統 | [Solution](solutions/object_oriented_design/call_center/call_center.ipynb)  |
| 設計一副牌 | [Solution](solutions/object_oriented_design/deck_of_cards/deck_of_cards.ipynb)  |
| 設計一個停車場 | [Solution](solutions/object_oriented_design/parking_lot/parking_lot.ipynb)  |
| 設計一個聊天室 | [Solution](solutions/object_oriented_design/online_chat/online_chat.ipynb)  |
| 設計一個環形陣列 | [Contribute](#contributing)  |
| 增加一個物件導向設計問題 | [Contribute](#contributing) |

## 系統設計主題：從這裡開始

你是系統設計的新手嗎？

首先，你需要對於基本的原則有一定的認識，知道他們是什麼，如何使用，以及他們的優缺點。

### 第一步：瀏覽關於可擴展性的影片

[哈佛大學可擴展性的影片](https://www.youtube.com/watch?v=-W9F__D3oY4)

* 包含以下主題：
    * 垂直擴展
    * 水平擴展
    * 快取
    * 負載平衡
    * 資料庫複寫
    * 資料庫分割

### 第二步：閱讀關於可擴展性的文章

[可擴展性](http://www.lecloud.net/tagged/scalability)

* 包含以下主題：
    * [複製](http://www.lecloud.net/post/7295452622/scalability-for-dummies-part-1-clones)
    * [資料庫](http://www.lecloud.net/post/7994751381/scalability-for-dummies-part-2-database)
    * [快取](http://www.lecloud.net/post/9246290032/scalability-for-dummies-part-3-cache)
    * [非同步](http://www.lecloud.net/post/9699762917/scalability-for-dummies-part-4-asynchronism)

### 下一步

接下來，我們需要看看某些取捨：

* **效能** vs **可擴展性**
* **延遲** vs **吞吐量**
* **可用性** vs **一致性**

記住，任何設計都是取捨。

接著，我們將會深入更具體的主題，包含 DNS、CDN 和負載平衡。

## 效能與可擴展性

如果服務**性能**的增加和資源的投入是成正比時，代表服務是可擴展的。一般來說，增加性能代表服務更多的工作單元，也可以處理更多的資料。<sup><a href="http://www.allthingsdistributed.com/2006/03/a_word_on_scalability.html">1</a></sup>

從另一方面來看看性能與可擴展性：

* 如果你的系統存在**性能**問題時，對單一使用者來說的感覺是慢的。
* 如果你的系統存在**可擴展性**問題時，對於單一使用者來說感覺較快，但在高負載的時候就會變慢。

### 來源及延伸閱讀

* [簡談可擴展性](http://www.allthingsdistributed.com/2006/03/a_word_on_scalability.html)
* [可擴展性、可用性、穩定性與相關模式](http://www.slideshare.net/jboner/scalability-availability-stability-patterns/)

## 延遲與吞吐量

**延遲** 指執行一個操作或運算結果所花費的時間。

**吞吐量** 指單位時間內執行此類型操作或運算的數量。

一般來說，你應該以**可接受的延遲**數量下的**最大化吞吐量**為設計目標。

### 來源及延伸閱讀

* [了解延遲與吞吐量](https://community.cadence.com/cadence_blogs_8/b/sd/archive/2010/09/13/understanding-latency-vs-throughput)

## 可用性與一致性

### CAP 理論

<p align="center">
  <img src="http://i.imgur.com/bgLMI2u.png">
  <br/>
  <i><a href=http://robertgreiner.com/2014/08/cap-theorem-revisited>來源：再看 CAP 理論</a></i>
</p>

在一個分散式系統中，只能滿足以下三個項目的任兩項：

* **一致性** - 每次讀取都可以得到最新的資料，但偶爾會拿到錯誤
* **可用性** - 每次讀取都可以得到非錯誤的回應，但不能保證可以得到最新的資料
* **部分容錯性** - 在任意分區的網路故障情況下，系統仍然能夠持續運行

*網路是不可靠的，你的設計必須要確保部分容錯性，所以你只能夠在一致性與可用性中做出取捨。*

#### CP - 一致性與部分容錯性

等待分區的節點回覆可能會導致超時錯誤，如果你的系統的需求是需要保證原子讀寫時，CP 是一個不錯的選擇。

#### AP - 可用性與部分容錯性

每個進行回覆的節點中的最新版本可能不是最新的，當分區節點解析完畢後，寫入的操作可能需要一些時間來傳播資料。

當你的系統需求需要保證 [最終一致性](#eventual-consistency)，或當外部系統故障時，系統要能夠繼續運作時，AP 是一個不錯的選擇。

### 來源及延伸閱讀

* [複習 CAP 理論](http://robertgreiner.com/2014/08/cap-theorem-revisited/)
* [簡單的介紹 CAP 理論](http://ksat.me/a-plain-english-introduction-to-cap-theorem/)
* [CAP 問與答](https://github.com/henryr/cap-faq)

## 一致性模式

當你的資料有多個副本時，要考慮怎麼同步他們，以便讓使用者有一致的資料顯示。想想 [CAP 理論](#cap-theorem)中的一致性定律 - 每次的訪問都可以得到最新的資料，但可能也會收到錯誤的回應。

### 弱一致性

在寫入之後，任何的存取不一定可以拿到資料，弱一致性將盡力確保能存取到最新的資料。

這種方式可以在 memcached 等系統中看到。弱一致性可以在 VoIP、視訊聊天或其他即時多人線上遊戲中看到相關的使用案例。比方說，如果你在通話中遺失幾秒鐘時間的資料，再重新連接後，你是無法聽到這幾秒鐘的內容。

### 最終一致性

在寫入後的讀取操作最終可以看到被寫入的資料(通常在數毫秒內)。資料透過非同步的方式被複製。

DNS 或是電子郵件系統使用的就是這種方式，最終一致性在高可用的系統中效果很好。

### 強一致性

在寫入後，讀取將立刻取得資料，資料是透過同步的方式寫入。

檔案系統或資料庫系統就是使用這種方式，強一致性在需要紀錄的系統中表現很好。

### 來源及延伸閱讀

* [資料中心的記錄行為](http://snarfed.org/transactions_across_datacenters_io.html)

## 可用性模式

關於可用性有兩種模式：**容錯轉移** 和 **複寫**。

### 容錯轉移

#### 主動到備用切換模式(AP Mode)

在這個模式下，heartbeat 訊號會在主動和備用的機器中發送，當 heartbeat 中斷時，備用的機器就會切換為主動機器的 IP 位置接替服務。

當機的時間取決於備用的機器是在「熱」待機狀態還是「冷」待機狀態。只有處於主動的機器會處理使用者來的流量。

這個模式的切換也被稱為主從的切換模式。

#### 雙主動切換模式(AA Mode)

在此模式下，兩台伺服器都會負責處理流量，流量會在他們之間進行分散負載。

如果是外部網路的伺服器，DNS 需要知道兩台機器的 IP 位置，如果是內部網路的伺服器，應用程式邏輯需要知道這兩台機器。

雙主動切換模式也被稱為 master-master 切換。

### 缺點：容錯轉移

* 容錯轉移會需要增加額外的硬體與複雜度。
* 如果在新寫入的資料被複製到備用的機器前系統就發生故障，那有可能會遺失資料。

### 複寫

#### 主動到備用複寫與雙主動複寫

This topic is further discussed in the [Database](#database) section:

這一個主題進一步討論了[資料庫](#資料庫)部分：

* [主動到備用複寫](#主動到備用複寫)
* [雙主動複寫](#雙主動複寫)

## 域名系統

<p align="center">
  <img src="http://i.imgur.com/IOyLj4i.jpg">
  <br/>
  <i><a href=http://www.slideshare.net/srikrupa5/dns-security-presentation-issa>資料來源：DNS 安全介紹</a></i>
</p>

DNS 是將域名轉換為 IP 地址的系統。

DNS 是階層式的架構，一部分的 DNS 伺服器位於頂層，當查詢域名時，你的路由器或 ISP 業者會提供連接到 DNS 伺服器的資訊。較底層的 DNS 伺服器會快取查詢的結果，而這些快取資訊會因為 DNS 的傳遞而逐漸更新。DNS 的結果可以暫存在瀏覽器或操作系統中一段時間，時間的長短取決於[存活時間(TTL)](https://en.wikipedia.org/wiki/Time_to_live)的設定。

* **NS 記錄 (域名伺服器)** - 指定解析域名或子域名的 DNS 伺服器。
* **MX 記錄 (電子郵件交換伺服器)** - 指定接收電子郵件的伺服器。
* **A 記錄 (地址)** - 指向要對應的 IP 位置。
* **CNAME (別名)** - 從一個域名指向另外一個域名，或是 `CNAME` (example.com 指向 www.example.com) 或指向一個 `A` 記錄。

[CloudFlare](https://www.cloudflare.com/dns/) 和[Route 53](https://aws.amazon.com/route53/) 提供了 DNS 的服務。而這些 DNS 服務商透過以下幾種方式來決定流量如何被分派：

* [加權輪詢](http://g33kinfo.com/info/archives/2657)
    * 防止流量進入正在維修中的伺服器
    * 在不同大小的集群中進行負載平衡
    * A/B 測試
* 基於延遲來路由請求
* 基於地理位置來路由請求

### DNS 的缺點

* 儘管可以透過快取來減輕 DNS 的延遲，但連接 DNS 伺服器還是帶來了些許的延遲。
* DNS 伺服器的管理是複雜的，儘管他通常由[政府、ISP 業者或大公司](http://superuser.com/questions/472695/who-controls-the-dns-servers/472729) 來處理。
* DNS 伺服器會有 [DDoS 攻擊](http://dyn.com/blog/dyn-analysis-summary-of-friday-october-21-attack/)，讓不知道 Twitter IP 的使用者無法訪問 Twitter 網站。

### 來源及延伸閱讀

* [DNS 架構](https://technet.microsoft.com/en-us/library/dd197427(v=ws.10).aspx)
* [維基百科](https://en.wikipedia.org/wiki/Domain_Name_System)
* [DNS 文章](https://support.dnsimple.com/categories/dns/)

## 內容傳遞網路(CDN)

<p align="center">
  <img src="http://i.imgur.com/h9TAuGI.jpg">
  <br/>
  <i><a href=https://www.creative-artworks.eu/why-use-a-content-delivery-network-cdn/>來源：為什麼要使用 CDN</a></i>
</p>

內容傳遞網路(CDN)是一種全球性的分散式代理伺服器，它透過靠近使用者的伺服器來提供檔案。通常 HTML/CSS/JS、圖片或影片等檔案會靜態檔案會透過 CDN 來提供，儘管 Amazon 的 CloudFront 也支援了動態內容的 CDN 服務。而 CDN 的 DNS 服務會告知使用者要連接哪一台伺服器。

透過 CDN 來取得檔案可以大幅度地增加請求的效率，因為：

* 從靠近使用者的伺服器來拿檔案
* 透過 CDN 來回應使用者，你的原始伺服器不需要處理請求

### 推送式 CDNs

當你的伺服器有檔案變動時，推送 CDN 會接收到新的變動內容，並重寫 URL 位置指向新的內容。你可以設定檔案內容什麼時候過期以及何時更新，檔案內容只有在變更或新增的時候才會推送，最小化流量，但最大化儲存。

流量較小的網站，或是內容不是經常更新的網站使用推送式的 CDN 相當適合，因為內容會被經常放置在 CDN 內，而不是常常需要重新抓取新檔案。

### 拉取式 CDNs

拉取式的 CDN 指的是當地一個使用者來請求該資源時，才從伺服器上抓取對應檔案。將檔案留在伺服器上並且重寫指向 CDN 的 URL，直到檔案被快取在 CDN 上為止，請求都會比較慢。

[存活時間 (TTL)](https://en.wikipedia.org/wiki/Time_to_live) 決定檔案要被緩存多久的時間。拉取式 CDN 可以節省儲存空間，但在過期的文件被更新之前，則會導致多餘的流量。

拉取式的 CDN 適合高流量的網站，因為檔案會被平均的分散在各個結點伺服器中。

### CDN 的缺點

* CDN 的成本取決於流量，在權衡評估後，你可能會因為成本而放棄使用。
* 如果在 TTL 過期之前就更新內容，CDN 的緩存內容可能會過期。
* 需要改變靜態內容的網址來指向 CDN。

### 來源及延伸閱讀

* [全球性的 CDN](http://repository.cmu.edu/cgi/viewcontent.cgi?article=2112&context=compsci)
* [拉取式和推拉式 CDN 的差別](http://www.travelblogadvice.com/technical/the-differences-between-push-and-pull-cdns/)
* [維基百科](https://en.wikipedia.org/wiki/Content_delivery_network)

## 負載平衡

<p align="center">
  <img src="http://i.imgur.com/h81n9iK.png">
  <br/>
  <i><a href=http://horicky.blogspot.com/2010/10/scalable-system-design-patterns.html>來源：可擴展的系統設計模式</a></i>
</p>

負載平衡將使用者的請求分發到後端伺服器和資料庫，不管是哪種情況，負載平衡器會將回應返回給對應的使用者。而負載平衡器之所以有效在於以下幾點：

* 避免請求被轉到非正常運作的伺服器
* 避免資源過載
* 避免單點失敗

負載平衡器可以透過硬體(較昂貴)或 HAProxy 等軟體來實現。

其餘額外的好處有：

* **SSL 終結** - 將傳入的請求解密，並且加密伺服器的回應，如此一來後端伺服器就不需要進行這些高度消耗資源的願算
    * 不需要在每一台機器上安裝 [X.509 憑證](https://en.wikipedia.org/wiki/X.509)。
* **Session 保存** - 發行 cookie，並將特定使用者的請求路由到同樣的後端伺服器上。

為了避免故障，通常會採用[active-passive](#active-passive)或[active-active](#active-active)這樣多個負載平衡器的模式。

負載平衡器會基於多種方法來路由請求：

* 隨機
* 最少負載
* Session/cookies
* [輪詢調度或加權輪詢調度](http://g33kinfo.com/info/archives/2657)
* [第四層負載平衡](#layer-4-load-balancing)
* [第七層負載平衡](#layer-7-load-balancing)

### 第四層負載平衡

第四層的負載平衡器會監看[傳輸層](#communication)的資訊來決定如何分發請求。一般來說，這包含了來源、目標 IP 位置，以及在 header 中的 port，但不包含資料本身的內容。第四層的負載平衡器會透過 [網路地址轉換(NAT)](https://www.nginx.com/resources/glossary/layer-4-load-balancing/)來向上游的伺服器轉發資料。

### 第七層負載平衡

第七層的負載平衡器會監看[應用層](#communication)來決定如何分發請求。這包含了請求的 header、訊息和 cookies。這種負載平衡器會終結網路的流量、讀取訊息並做出如何轉發訊息的決定，並把流量轉往對應的伺服器。舉例來說，一個第七層的負載平衡器可以將影音的流量轉往負責影音流量的伺服器，而將更敏感的使用者帳單的請求轉往安全性更強的伺服器。

第四層的負載平衡比起第七層的所要花費的時間和計算資源更低，雖然這對於現代商用硬體的性能影響已經微乎其微了。

### 水平擴展

負載平衡器一樣可以幫助水平擴展，提高性能與可用性。使用這種方式的擴展比起在單一機器的**垂直擴展**來說性價比更高，同時，聘請商用硬體的人才比起特定企業級系統人才來的更加容易。

#### 水平擴展的缺點

* 水平擴展會增加複雜性，同時也涉及了多台伺服器的議題
    * 伺服器應該是無狀態的：不應該包括像是 session 或資料圖片等和使用者相關的內容
    * Session 可以集中儲存在資料庫或[快取](#快取)(Redis、Memcached) 等資料儲存中。
* 快取伺服器或資料庫需要隨著伺服器的增加而進行擴展，以便處理更多的請求。

### 負載平衡器的缺點

* 當負載平衡器資源不夠或沒有正確設定時，他可能會成為效能的瓶頸
* 使用負載平衡器來避免單點失敗會增加架構的複雜性
* 只有一台負載平衡器時，一樣有單點失敗的問題。而多台的負載平衡器一樣增加了架構的複雜性。

### 來源及延伸閱讀

* [NGINX 架構](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/)
* [HAProxy 架構指南](http://www.haproxy.org/download/1.2/doc/architecture.txt)
* [可擴展性](http://www.lecloud.net/post/7295452622/scalability-for-dummies-part-1-clones)
* [維基百科](https://en.wikipedia.org/wiki/Load_balancing_(computing))
* [第四層負載平衡](https://www.nginx.com/resources/glossary/layer-4-load-balancing/)
* [第七層負載平衡](https://www.nginx.com/resources/glossary/layer-7-load-balancing/)
* [ELB 監聽器設定](http://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-listener-config.html)

## 反向代理 (網頁伺服器)

<p align="center">
  <img src="http://i.imgur.com/n41Azff.png">
  <br/>
  <i><a href=https://upload.wikimedia.org/wikipedia/commons/6/67/Reverse_proxy_h2g2bob.svg>來源：維基百科</a></i>
  <br/>
</p>

反向代理伺服器是一個集中內部服務，並提供統一個介面給公開使用者的伺服器。來自客戶端的請求會先被反向代理伺服器轉發到可以接收服務的伺服器，然後再由代理伺服器將結果返回給客戶端。

這樣做的好處有：

* **增加安全性** - 隱藏後端伺服器的資訊、可以設定 IP 的黑名單、限制每個客戶端的連線數量等。
* **增加可擴展性與靈活性** - 客戶端只會看到反向代理伺服器的 IP 或域名，這樣你就可以增加背後伺服器的數量或設定而不影響客戶端。
* **SSL 終止** - 解密傳入的請求、加密伺服器的回應，這樣後端伺服器就不需要進行這些高成本的操作
    * 不需要在每一台伺服器安裝 [X.509 憑證](https://en.wikipedia.org/wiki/X.509)。
* **壓縮** - 壓縮伺服器的回應
* **快取** - 直接在代理伺服器回應命中快取的結果
* **靜態檔案** - 直接提供靜態內容
    * HTML/CSS/JS
    * 圖片
    * 影片
    * 等等

### 負載平衡器與反向代理伺服器

* 當有多台伺服器時，使用負載平衡非常有用，一般來說，負載平衡器會將流量路由給一組功能相同的伺服器上。
* 即使只有一台伺服器或應用伺服器，反向代理也是有用的。可以參考上述的好處。
* Nginx 或 HAProxy 等解決方案可以同時支援第七層的反向代理與負載平衡

### 反向代理伺服器的缺點

* 引入反向代理伺服器會增加系統複雜度。
* 只有一台反向代理伺服器會有單點失效的問題，而設定多台的反向代理伺服器(如[故障轉移](https://en.wikipedia.org/wiki/Failover))同樣會增加系統複雜度。

### 來源與延伸閱讀

* [反向代理伺服器與負載平衡](https://www.nginx.com/resources/glossary/reverse-proxy-vs-load-balancer/)
* [NGINX 架構](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/)
* [HAProxy 架構指南](http://www.haproxy.org/download/1.2/doc/architecture.txt)
* [維基百科](https://en.wikipedia.org/wiki/Reverse_proxy)

## Application layer

<p align="center">
  <img src="http://i.imgur.com/yB5SYwm.png">
  <br/>
  <i><a href=http://lethain.com/introduction-to-architecting-systems-for-scale/#platform_layer>Source: Intro to architecting systems for scale</a></i>
</p>

Separating out the web layer from the application layer (also known as platform layer) allows you to scale and configure both layers independently.  Adding a new API results in adding application servers without necessarily adding additional web servers.

The **single responsibility principle** advocates for small and autonomous services that work together.  Small teams with small services can plan more aggressively for rapid growth.

Workers in the application layer also help enable [asynchronism](#asynchronism).

### Microservices

Related to this discussion are [microservices](https://en.wikipedia.org/wiki/Microservices), which can be described as a suite of independently deployable, small, modular services.  Each service runs a unique process and communicates through a well-defined, lightweight mechanism to serve a business goal. <sup><a href=https://smartbear.com/learn/api-design/what-are-microservices>1</a></sup>

Pinterest, for example, could have the following microservices: user profile, follower, feed, search, photo upload, etc.

### Service Discovery

Systems such as [Consul](https://www.consul.io/docs/index.html), [Etcd](https://coreos.com/etcd/docs/latest), and [Zookeeper](http://www.slideshare.net/sauravhaloi/introduction-to-apache-zookeeper) can help services find each other by keeping track of registered names, addresses, and ports.  [Health checks](https://www.consul.io/intro/getting-started/checks.html) help verify service integrity and are often done using an [HTTP](#hypertext-transfer-protocol-http) endpoint.  Both Consul and Etcd have a built in [key-value store](#key-value-store) that can be useful for storing config values and other shared data.

### Disadvantage(s): application layer

* Adding an application layer with loosely coupled services requires a different approach from an architectural, operations, and process viewpoint (vs a monolithic system).
* Microservices can add complexity in terms of deployments and operations.

### Source(s) and further reading

* [Intro to architecting systems for scale](http://lethain.com/introduction-to-architecting-systems-for-scale)
* [Crack the system design interview](http://www.puncsky.com/blog/2016/02/14/crack-the-system-design-interview/)
* [Service oriented architecture](https://en.wikipedia.org/wiki/Service-oriented_architecture)
* [Introduction to Zookeeper](http://www.slideshare.net/sauravhaloi/introduction-to-apache-zookeeper)
* [Here's what you need to know about building microservices](https://cloudncode.wordpress.com/2016/07/22/msa-getting-started/)

## Database

<p align="center">
  <img src="http://i.imgur.com/Xkm5CXz.png">
  <br/>
  <i><a href=https://www.youtube.com/watch?v=vg5onp8TU6Q>Source: Scaling up to your first 10 million users</a></i>
</p>

### Relational database management system (RDBMS)

A relational database like SQL is a collection of data items organized in tables.

**ACID** is a set of properties of relational database [transactions](https://en.wikipedia.org/wiki/Database_transaction).

* **Atomicity** - Each transaction is all or nothing
* **Consistency** - Any transaction will bring the database from one valid state to another
* **Isolation** - Executing transactions concurrently has the same results as if the transactions were executed serially
* **Durability** - Once a transaction has been committed, it will remain so

There are many techniques to scale a relational database: **master-slave replication**, **master-master replication**, **federation**, **sharding**, **denormalization**, and **SQL tuning**.

#### Master-slave replication

The master serves reads and writes, replicating writes to one or more slaves, which serve only reads.  Slaves can also replicate to additional slaves in a tree-like fashion.  If the master goes offline, the system can continue to operate in read-only mode until a slave is promoted to a master or a new master is provisioned.

<p align="center">
  <img src="http://i.imgur.com/C9ioGtn.png">
  <br/>
  <i><a href=http://www.slideshare.net/jboner/scalability-availability-stability-patterns/>Source: Scalability, availability, stability, patterns</a></i>
</p>

##### Disadvantage(s): master-slave replication

* Additional logic is needed to promote a slave to a master.
* See [Disadvantage(s): replication](#disadvantages-replication) for points related to **both** master-slave and master-master.

#### Master-master replication

Both masters serve reads and writes and coordinate with each other on writes.  If either master goes down, the system can continue to operate with both reads and writes.

<p align="center">
  <img src="http://i.imgur.com/krAHLGg.png">
  <br/>
  <i><a href=http://www.slideshare.net/jboner/scalability-availability-stability-patterns/>Source: Scalability, availability, stability, patterns</a></i>
</p>

##### Disadvantage(s): master-master replication

* You'll need a load balancer or you'll need to make changes to your application logic to determine where to write.
* Most master-master systems are either loosely consistent (violating ACID) or have increased write latency due to synchronization.
* Conflict resolution comes more into play as more write nodes are added and as latency increases.
* See [Disadvantage(s): replication](#disadvantages-replication) for points related to **both** master-slave and master-master.

##### Disadvantage(s): replication

* There is a potential for loss of data if the master fails before any newly written data can be replicated to other nodes.
* Writes are replayed to the read replicas.  If there are a lot of writes, the read replicas can get bogged down with replaying writes and can't do as many reads.
* The more read slaves, the more you have to replicate, which leads to greater replication lag.
* On some systems, writing to the master can spawn multiple threads to write in parallel, whereas read replicas only support writing sequentially with a single thread.
* Replication adds more hardware and additional complexity.

##### Source(s) and further reading: replication

* [Scalability, availability, stability, patterns](http://www.slideshare.net/jboner/scalability-availability-stability-patterns/)
* [Multi-master replication](https://en.wikipedia.org/wiki/Multi-master_replication)

#### Federation

<p align="center">
  <img src="http://i.imgur.com/U3qV33e.png">
  <br/>
  <i><a href=https://www.youtube.com/watch?v=vg5onp8TU6Q>Source: Scaling up to your first 10 million users</a></i>
</p>

Federation (or functional partitioning) splits up databases by function.  For example, instead of a single, monolithic database, you could have three databases: **forums**, **users**, and **products**, resulting in less read and write traffic to each database and therefore less replication lag.  Smaller databases result in more data that can fit in memory, which in turn results in more cache hits due to improved cache locality.  With no single central master serializing writes you can write in parallel, increasing throughput.

##### Disadvantage(s): federation

* Federation is not effective if your schema requires huge functions or tables.
* You'll need to update your application logic to determine which database to read and write.
* Joining data from two databases is more complex with a [server link](http://stackoverflow.com/questions/5145637/querying-data-by-joining-two-tables-in-two-database-on-different-servers).
* Federation adds more hardware and additional complexity.

##### Source(s) and further reading: federation

* [Scaling up to your first 10 million users](https://www.youtube.com/watch?v=vg5onp8TU6Q)

#### Sharding

<p align="center">
  <img src="http://i.imgur.com/wU8x5Id.png">
  <br/>
  <i><a href=http://www.slideshare.net/jboner/scalability-availability-stability-patterns/>Source: Scalability, availability, stability, patterns</a></i>
</p>

Sharding distributes data across different databases such that each database can only manage a subset of the data.  Taking a users database as an example, as the number of users increases, more shards are added to the cluster.

Similar to the advantages of [federation](#federation), sharding results in less read and write traffic, less replication, and more cache hits.  Index size is also reduced, which generally improves performance with faster queries.  If one shard goes down, the other shards are still operational, although you'll want to add some form of replication to avoid data loss.  Like federation, there is no single central master serializing writes, allowing you to write in parallel with increased throughput.

Common ways to shard a table of users is either through the user's last name initial or the user's geographic location.

##### Disadvantage(s): sharding

* You'll need to update your application logic to work with shards, which could result in complex SQL queries.
* Data distribution can become lopsided in a shard.  For example, a set of power users on a shard could result in increased load to that shard compared to others.
    * Rebalancing adds additional complexity.  A sharding function based on [consistent hashing](http://www.paperplanes.de/2011/12/9/the-magic-of-consistent-hashing.html) can reduce the amount of transferred data.
* Joining data from multiple shards is more complex.
* Sharding adds more hardware and additional complexity.

##### Source(s) and further reading: sharding

* [The coming of the shard](http://highscalability.com/blog/2009/8/6/an-unorthodox-approach-to-database-design-the-coming-of-the.html)
* [Shard database architecture](https://en.wikipedia.org/wiki/Shard_(database_architecture))
* [Consistent hashing](http://www.paperplanes.de/2011/12/9/the-magic-of-consistent-hashing.html)

#### Denormalization

Denormalization attempts to improve read performance at the expense of some write performance.  Redundant copies of the data are written in multiple tables to avoid expensive joins.  Some RDBMS such as [PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL) and Oracle support [materialized views](https://en.wikipedia.org/wiki/Materialized_view) which handle the work of storing redundant information and keeping redundant copies consistent.

Once data becomes distributed with techniques such as [federation](#federation) and [sharding](#sharding), managing joins across data centers further increases complexity.  Denormalization might circumvent the need for such complex joins.

In most systems, reads can heavily number writes 100:1 or even 1000:1.  A read resulting in a complex database join can be very expensive, spending a significant amount of time on disk operations.

##### Disadvantage(s): denormalization

* Data is duplicated.
* Constraints can help redundant copies of information stay in sync, which increases complexity of the database design.
* A denormalized database under heavy write load might perform worse than its normalized counterpart.

###### Source(s) and further reading: denormalization

* [Denormalization](https://en.wikipedia.org/wiki/Denormalization)

#### SQL tuning

SQL tuning is a broad topic and many [books](https://www.amazon.com/s/ref=nb_sb_noss_2?url=search-alias%3Daps&field-keywords=sql+tuning) have been written as reference.

It's important to **benchmark** and **profile** to simulate and uncover bottlenecks.

* **Benchmark** - Simulate high-load situations with tools such as [ab](http://httpd.apache.org/docs/2.2/programs/ab.html).
* **Profile** - Enable tools such as the [slow query log](http://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html) to help track performance issues.

Benchmarking and profiling might point you to the following optimizations.

##### Tighten up the schema

* MySQL dumps to disk in contiguous blocks for fast access.
* Use `CHAR` instead of `VARCHAR` for fixed-length fields.
    * `CHAR` effectively allows for fast, random access, whereas with `VARCHAR`, you must find the end of a string before moving onto the next one.
* Use `TEXT` for large blocks of text such as blog posts.  `TEXT` also allows for boolean searches.  Using a `TEXT` field results in storing a pointer on disk that is used to locate the text block.
* Use `INT` for larger numbers up to 2^32 or 4 billion.
* Use `DECIMAL` for currency to avoid floating point representation errors.
* Avoid storing large `BLOBS`, store the location of where to get the object instead.
* `VARCHAR(255)` is the largest number of characters that can be counted in an 8 bit number, often maximizing the use of a byte in some RDBMS.
* Set the `NOT NULL` constraint where applicable to [improve search performance](http://stackoverflow.com/questions/1017239/how-do-null-values-affect-performance-in-a-database-search).

##### Use good indices

* Columns that you are querying (`SELECT`, `GROUP BY`, `ORDER BY`, `JOIN`) could be faster with indices.
* Indices are usually represented as self-balancing [B-tree](https://en.wikipedia.org/wiki/B-tree) that keeps data sorted and allows searches, sequential access, insertions, and deletions in logarithmic time.
* Placing an index can keep the data in memory, requiring more space.
* Writes could also be slower since the index also needs to be updated.
* When loading large amounts of data, it might be faster to disable indices, load the data, then rebuild the indices.

##### Avoid expensive joins

* [Denormalize](#denormalization) where performance demands it.

##### Partition tables

* Break up a table by putting hot spots in a separate table to help keep it in memory.

##### Tune the query cache

* In some cases, the [query cache](http://dev.mysql.com/doc/refman/5.7/en/query-cache) could lead to [performance issues](https://www.percona.com/blog/2014/01/28/10-mysql-performance-tuning-settings-after-installation/).

##### Source(s) and further reading: SQL tuning

* [Tips for optimizing MySQL queries](http://20bits.com/article/10-tips-for-optimizing-mysql-queries-that-dont-suck)
* [Is there a good reason i see VARCHAR(255) used so often?](http://stackoverflow.com/questions/1217466/is-there-a-good-reason-i-see-varchar255-used-so-often-as-opposed-to-another-l)
* [How do null values affect performance?](http://stackoverflow.com/questions/1017239/how-do-null-values-affect-performance-in-a-database-search)
* [Slow query log](http://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html)

### NoSQL

NoSQL is a collection of data items represented in a **key-value store**, **document-store**, **wide column store**, or a **graph database**.  Data is denormalized, and joins are generally done in the application code.  Most NoSQL stores lack true ACID transactions and favor [eventual consistency](#eventual-consistency).

**BASE** is often used to describe the properties of NoSQL databases.  In comparison with the [CAP Theorem](#cap-theorem), BASE chooses availability over consistency.

* **Basically available** - the system guarantees availability.
* **Soft state** - the state of the system may change over time, even without input.
* **Eventual consistency** - the system will become consistent over a period of time, given that the system doesn't receive input during that period.

In addition to choosing between [SQL or NoSQL](#sql-or-nosql), it is helpful to understand which type of NoSQL database best fits your use case(s).  We'll review **key-value stores**, **document-stores**, **wide column stores**, and **graph databases** in the next section.

#### Key-value store

> Abstraction: hash table

A key-value store generally allows for O(1) reads and writes and is often backed by memory or SSD.  Data stores can maintain keys in [lexicographic order](https://en.wikipedia.org/wiki/Lexicographical_order), allowing efficient retrieval of key ranges.  Key-value stores can allow for storing of metadata with a value.

Key-value stores provide high performance and are often used for simple data models or for rapidly-changing data, such as an in-memory cache layer.  Since they offer only a limited set of operations, complexity is shifted to the application layer if additional operations are needed.

A key-value store is the basis for more complex systems such as a document store, and in some cases, a graph database.

##### Source(s) and further reading: key-value store

* [Key-value database](https://en.wikipedia.org/wiki/Key-value_database)
* [Disadvantages of key-value stores](http://stackoverflow.com/questions/4056093/what-are-the-disadvantages-of-using-a-key-value-table-over-nullable-columns-or)
* [Redis architecture](http://qnimate.com/overview-of-redis-architecture/)
* [Memcached architecture](https://www.adayinthelifeof.nl/2011/02/06/memcache-internals/)

#### Document store

> Abstraction: key-value store with documents stored as values

A document store is centered around documents (XML, JSON, binary, etc), where a document stores all information for a given object.  Document stores provide APIs or a query language to query based on the internal structure of the document itself.  *Note, many key-value stores include features for working with a value's metadata, blurring the lines between these two storage types.*

Based on the underlying implementation, documents are organized in either collections, tags, metadata, or directories.  Although documents can be organized or grouped together, documents may have fields that are completely different from each other.

Some document stores like [MongoDB](https://www.mongodb.com/mongodb-architecture) and [CouchDB](https://blog.couchdb.org/2016/08/01/couchdb-2-0-architecture/) also provide a SQL-like language to perform complex queries.  [DynamoDB](http://www.read.seas.harvard.edu/~kohler/class/cs239-w08/decandia07dynamo.pdf) supports both key-values and documents.

Document stores provide high flexibility and are often used for working with occasionally changing data.

##### Source(s) and further reading: document store

* [Document-oriented database](https://en.wikipedia.org/wiki/Document-oriented_database)
* [MongoDB architecture](https://www.mongodb.com/mongodb-architecture)
* [CouchDB architecture](https://blog.couchdb.org/2016/08/01/couchdb-2-0-architecture/)
* [Elasticsearch architecture](https://www.elastic.co/blog/found-elasticsearch-from-the-bottom-up)

#### Wide column store

<p align="center">
  <img src="http://i.imgur.com/n16iOGk.png">
  <br/>
  <i><a href=http://blog.grio.com/2015/11/sql-nosql-a-brief-history.html>Source: SQL & NoSQL, a brief history</a></i>
</p>

> Abstraction: nested map `ColumnFamily<RowKey, Columns<ColKey, Value, Timestamp>>`

A wide column store's basic unit of data is a column (name/value pair).  A column can be grouped in column families (analogous to a SQL table).  Super column families further group column families.  You can access each column independently with a row key, and columns with the same row key form a row.  Each value contains a timestamp for versioning and for conflict resolution.

Google introduced [Bigtable](http://www.read.seas.harvard.edu/~kohler/class/cs239-w08/chang06bigtable.pdf) as the first wide column store, which influenced the open-source [HBase](https://www.mapr.com/blog/in-depth-look-hbase-architecture) often-used in the Hadoop ecosystem, and [Cassandra](http://docs.datastax.com/en/archived/cassandra/2.0/cassandra/architecture/architectureIntro_c.html) from Facebook.  Stores such as BigTable, HBase, and Cassandra maintain keys in lexicographic order, allowing efficient retrieval of selective key ranges.

Wide column stores offer high availability and high scalability.  They are often used for very large data sets.

##### Source(s) and further reading: wide column store

* [SQL & NoSQL, a brief history](http://blog.grio.com/2015/11/sql-nosql-a-brief-history.html)
* [Bigtable architecture](http://www.read.seas.harvard.edu/~kohler/class/cs239-w08/chang06bigtable.pdf)
* [HBase architecture](https://www.mapr.com/blog/in-depth-look-hbase-architecture)
* [Cassandra architecture](http://docs.datastax.com/en/archived/cassandra/2.0/cassandra/architecture/architectureIntro_c.html)

#### Graph database

<p align="center">
  <img src="http://i.imgur.com/fNcl65g.png">
  <br/>
  <i><a href=https://en.wikipedia.org/wiki/File:GraphDatabase_PropertyGraph.png>Source: Graph database</a></i>
</p>

> Abstraction: graph

In a graph database, each node is a record and each arc is a relationship between two nodes.  Graph databases are optimized to represent complex relationships with many foreign keys or many-to-many relationships.

Graphs databases offer high performance for data models with complex relationships, such as a social network.  They are relatively new and are not yet widely-used; it might be more difficult to find development tools and resources.  Many graphs can only be accessed with [REST APIs](#representational-state-transfer-rest).

##### Source(s) and further reading: graph

* [Graph database](https://en.wikipedia.org/wiki/Graph_database)
* [Neo4j](https://neo4j.com/)
* [FlockDB](https://blog.twitter.com/2010/introducing-flockdb)

#### Source(s) and further reading: NoSQL

* [Explanation of base terminology](http://stackoverflow.com/questions/3342497/explanation-of-base-terminology)
* [NoSQL databases a survey and decision guidance](https://medium.com/baqend-blog/nosql-databases-a-survey-and-decision-guidance-ea7823a822d#.wskogqenq)
* [Scalability](http://www.lecloud.net/post/7994751381/scalability-for-dummies-part-2-database)
* [Introduction to NoSQL](https://www.youtube.com/watch?v=qI_g07C_Q5I)
* [NoSQL patterns](http://horicky.blogspot.com/2009/11/nosql-patterns.html)

### SQL or NoSQL

<p align="center">
  <img src="http://i.imgur.com/wXGqG5f.png">
  <br/>
  <i><a href=https://www.infoq.com/articles/Transition-RDBMS-NoSQL/>Source: Transitioning from RDBMS to NoSQL</a></i>
</p>

Reasons for **SQL**:

* Structured data
* Strict schema
* Relational data
* Need for complex joins
* Transactions
* Clear patterns for scaling
* More established: developers, community, code, tools, etc
* Lookups by index are very fast

Reasons for **NoSQL**:

* Semi-structured data
* Dynamic or flexible schema
* Non-relational data
* No need for complex joins
* Store many TB (or PB) of data
* Very data intensive workload
* Very high throughput for IOPS

Sample data well-suited for NoSQL:

* Rapid ingest of clickstream and log data
* Leaderboard or scoring data
* Temporary data, such as a shopping cart
* Frequently accessed ('hot') tables
* Metadata/lookup tables

##### Source(s) and further reading: SQL or NoSQL

* [Scaling up to your first 10 million users](https://www.youtube.com/watch?v=vg5onp8TU6Q)
* [SQL vs NoSQL differences](https://www.sitepoint.com/sql-vs-nosql-differences/)

## Cache

<p align="center">
  <img src="http://i.imgur.com/Q6z24La.png">
  <br/>
  <i><a href=http://horicky.blogspot.com/2010/10/scalable-system-design-patterns.html>Source: Scalable system design patterns</a></i>
</p>

Caching improves page load times and can reduce the load on your servers and databases.  In this model, the dispatcher will first lookup if the request has been made before and try to find the previous result to return, in order to save the actual execution.

Databases often benefit from a uniform distribution of reads and writes across its partitions.  Popular items can skew the distribution, causing bottlenecks.  Putting a cache in front of a database can help absorb uneven loads and spikes in traffic.

### Client caching

Caches can be located on the client side (OS or browser), [server side](#reverse-proxy), or in a distinct cache layer.

### CDN caching

[CDNs](#content-delivery-network) are considered a type of cache.

### Web server caching

[Reverse proxies](#reverse-proxy-web-server) and caches such as [Varnish](https://www.varnish-cache.org/) can serve static and dynamic content directly.  Web servers can also cache requests, returning responses without having to contact application servers.

### Database caching

Your database usually includes some level of caching in a default configuration, optimized for a generic use case.  Tweaking these settings for specific usage patterns can further boost performance.

### Application caching

In-memory caches such as Memcached and Redis are key-value stores between your application and your data storage.  Since the data is held in RAM, it is much faster than typical databases where data is stored on disk.  RAM is more limited than disk, so [cache invalidation](https://en.wikipedia.org/wiki/Cache_algorithms) algorithms such as [least recently used (LRU)](https://en.wikipedia.org/wiki/Cache_algorithms#Least_Recently_Used) can help invalidate 'cold' entries and keep 'hot' data in RAM.

Redis has the following additional features:

* Persistence option
* Built-in data structures such as sorted sets and lists

There are multiple levels you can cache that fall into two general categories: **database queries** and **objects**:

* Row level
* Query-level
* Fully-formed serializable objects
* Fully-rendered HTML

Generally, you should try to avoid file-based caching, as it makes cloning and auto-scaling more difficult.

### Caching at the database query level

Whenever you query the database, hash the query as a key and store the result to the cache.  This approach suffers from expiration issues:

* Hard to delete a cached result with complex queries
* If one piece of data changes such as a table cell, you need to delete all cached queries that might include the changed cell

### Caching at the object level

See your data as an object, similar to what you do with your application code.  Have your application assemble the dataset from the database into a class instance or a data structure(s):

* Remove the object from cache if its underlying data has changed
* Allows for asynchronous processing: workers assemble objects by consuming the latest cached object

Suggestions of what to cache:

* User sessions
* Fully rendered web pages
* Activity streams
* User graph data

### When to update the cache

Since you can only store a limited amount of data in cache, you'll need to determine which cache update strategy works best for your use case.

#### Cache-aside

<p align="center">
  <img src="http://i.imgur.com/ONjORqk.png">
  <br/>
  <i><a href=http://www.slideshare.net/tmatyashovsky/from-cache-to-in-memory-data-grid-introduction-to-hazelcast>Source: From cache to in-memory data grid</a></i>
</p>

The application is responsible for reading and writing from storage.  The cache does not interact with storage directly.  The application does the following:

* Look for entry in cache, resulting in a cache miss
* Load entry from the database
* Add entry to cache
* Return entry

```
def get_user(self, user_id):
    user = cache.get("user.{0}", user_id)
    if user is None:
        user = db.query("SELECT * FROM users WHERE user_id = {0}", user_id)
        if user is not None:
            key = "user.{0}".format(user_id)
            cache.set(key, json.dumps(user))
    return user
```

[Memcached](https://memcached.org/) is generally used in this manner.

Subsequent reads of data added to cache are fast.  Cache-aside is also referred to as lazy loading.  Only requested data is cached, which avoids filling up the cache with data that isn't requested.

##### Disadvantage(s): cache-aside

* Each cache miss results in three trips, which can cause a noticeable delay.
* Data can become stale if it is updated in the database.  This issue is mitigated by setting a time-to-live (TTL) which forces an update of the cache entry, or by using write-through.
* When a node fails, it is replaced by a new, empty node, increasing latency.

#### Write-through

<p align="center">
  <img src="http://i.imgur.com/0vBc0hN.png">
  <br/>
  <i><a href=http://www.slideshare.net/jboner/scalability-availability-stability-patterns/>Source: Scalability, availability, stability, patterns</a></i>
</p>

The application uses the cache as the main data store, reading and writing data to it, while the cache is responsible for reading and writing to the database:

* Application adds/updates entry in cache
* Cache synchronously writes entry to data store
* Return

Application code:

```
set_user(12345, {"foo":"bar"})
```

Cache code:

```
def set_user(user_id, values):
    user = db.query("UPDATE Users WHERE id = {0}", user_id, values)
    cache.set(user_id, user)
```

Write-through is a slow overall operation due to the write operation, but subsequent reads of just written data are fast.  Users are generally more tolerant of latency when updating data than reading data.  Data in the cache is not stale.

##### Disadvantage(s): write through

* When a new node is created due to failure or scaling, the new node will not cache entries until the entry is updated in the database.  Cache-aside in conjunction with write through can mitigate this issue.
* Most data written might never read, which can be minimized with a TTL.

#### Write-behind (write-back)

<p align="center">
  <img src="http://i.imgur.com/rgSrvjG.png">
  <br/>
  <i><a href=http://www.slideshare.net/jboner/scalability-availability-stability-patterns/>Source: Scalability, availability, stability, patterns</a></i>
</p>

In write-behind, the application does the following:

* Add/update entry in cache
* Asynchronously write entry to the data store, improving write performance

##### Disadvantage(s): write-behind

* There could be data loss if the cache goes down prior to its contents hitting the data store.
* It is more complex to implement write-behind than it is to implement cache-aside or write-through.

#### Refresh-ahead

<p align="center">
  <img src="http://i.imgur.com/kxtjqgE.png">
  <br/>
  <i><a href=http://www.slideshare.net/tmatyashovsky/from-cache-to-in-memory-data-grid-introduction-to-hazelcast>Source: From cache to in-memory data grid</a></i>
</p>

You can configure the cache to automatically refresh any recently accessed cache entry prior to its expiration.

Refresh-ahead can result in reduced latency vs read-through if the cache can accurately predict which items are likely to be needed in the future.

##### Disadvantage(s): refresh-ahead

* Not accurately predicting which items are likely to be needed in the future can result in reduced performance than without refresh-ahead.

### Disadvantage(s): cache

* Need to maintain consistency between caches and the source of truth such as the database through [cache invalidation](https://en.wikipedia.org/wiki/Cache_algorithms).
* Need to make application changes such as adding Redis or memcached.
* Cache invalidation is a difficult problem, there is additional complexity associated with when to update the cache.

### Source(s) and further reading

* [From cache to in-memory data grid](http://www.slideshare.net/tmatyashovsky/from-cache-to-in-memory-data-grid-introduction-to-hazelcast)
* [Scalable system design patterns](http://horicky.blogspot.com/2010/10/scalable-system-design-patterns.html)
* [Introduction to architecting systems for scale](http://lethain.com/introduction-to-architecting-systems-for-scale/)
* [Scalability, availability, stability, patterns](http://www.slideshare.net/jboner/scalability-availability-stability-patterns/)
* [Scalability](http://www.lecloud.net/post/9246290032/scalability-for-dummies-part-3-cache)
* [AWS ElastiCache strategies](http://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/Strategies.html)
* [Wikipedia](https://en.wikipedia.org/wiki/Cache_(computing))

## Asynchronism

<p align="center">
  <img src="http://i.imgur.com/54GYsSx.png">
  <br/>
  <i><a href=http://lethain.com/introduction-to-architecting-systems-for-scale/#platform_layer>Source: Intro to architecting systems for scale</a></i>
</p>

Asynchronous workflows help reduce request times for expensive operations that would otherwise be performed in-line.  They can also help by doing time-consuming work in advance, such as periodic aggregation of data.

### Message queues

Message queues receive, hold, and deliver messages.  If an operation is too slow to perform inline, you can use a message queue with the following workflow:

* An application publishes a job to the queue, then notifies the user of job status
* A worker picks up the job from the queue, processes it, then signals the job is complete

The user is not blocked and the job is processed in the background.  During this time, the client might optionally do a small amount of processing to make it seem like the task has completed.  For example, if posting a tweet, the tweet could be instantly posted to your timeline, but it could take some time before your tweet is actually delivered to all of your followers.

**Redis** is useful as a simple message broker but messages can be lost.

**RabbitMQ** is popular but requires you to adapt to the 'AMQP' protocol and manage your own nodes.

**Amazon SQS**, is hosted but can have high latency and has the possibility of messages being delivered twice.

### Task queues

Tasks queues receive tasks and their related data, runs them, then delivers their results.  They can support scheduling and can be used to run computationally-intensive jobs in the background.

**Celery** has support for scheduling and primarily has python support.

### Back pressure

If queues start to grow significantly, the queue size can become larger than memory, resulting in cache misses, disk reads, and even slower performance.  [Back pressure](http://mechanical-sympathy.blogspot.com/2012/05/apply-back-pressure-when-overloaded.html) can help by limiting the queue size, thereby maintaining a high throughput rate and good response times for jobs already in the queue.  Once the queue fills up, clients get a server busy or HTTP 503 status code to try again later.  Clients can retry the request at a later time, perhaps with [exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff).

### Disadvantage(s): asynchronism

* Use cases such as inexpensive calculations and realtime workflows might be better suited for synchronous operations, as introducing queues can add delays and complexity.

### Source(s) and further reading

* [It's all a numbers game](https://www.youtube.com/watch?v=1KRYH75wgy4)
* [Applying back pressure when overloaded](http://mechanical-sympathy.blogspot.com/2012/05/apply-back-pressure-when-overloaded.html)
* [Little's law](https://en.wikipedia.org/wiki/Little%27s_law)
* [What is the difference between a message queue and a task queue?](https://www.quora.com/What-is-the-difference-between-a-message-queue-and-a-task-queue-Why-would-a-task-queue-require-a-message-broker-like-RabbitMQ-Redis-Celery-or-IronMQ-to-function)

## Communication

<p align="center">
  <img src="http://i.imgur.com/5KeocQs.jpg">
  <br/>
  <i><a href=http://www.escotal.com/osilayer.html>Source: OSI 7 layer model</a></i>
</p>

### Hypertext transfer protocol (HTTP)

HTTP is a method for encoding and transporting data between a client and a server.  It is a request/response protocol: clients issue requests and servers issue responses with relevant content and completion status info about the request.  HTTP is self-contained, allowing requests and responses to flow through many intermediate routers and servers that perform load balancing, caching, encryption, and compression.

A basic HTTP request consists of a verb (method) and a resource (endpoint).  Below are common HTTP verbs:

| Verb | Description | Idempotent* | Safe | Cacheable |
|---|---|---|---|---|
| GET | Reads a resource | Yes | Yes | Yes |
| POST | Creates a resource or trigger a process that handles data | No | No | Yes if response contains freshness info |
| PUT | Creates or replace a resource | Yes | No | No |
| PATCH | Partially updates a resource | No | No | Yes if response contains freshness info |
| DELETE | Deletes a resource | Yes | No | No |

*Can be called many times without different outcomes.

HTTP is an application layer protocol relying on lower-level protocols such as **TCP** and **UDP**.

#### Source(s) and further reading: HTTP

* [What is HTTP?](https://www.nginx.com/resources/glossary/http/)
* [Difference between HTTP and TCP](https://www.quora.com/What-is-the-difference-between-HTTP-protocol-and-TCP-protocol)
* [Difference between PUT and PATCH](https://laracasts.com/discuss/channels/general-discussion/whats-the-differences-between-put-and-patch?page=1)

### Transmission control protocol (TCP)

<p align="center">
  <img src="http://i.imgur.com/JdAsdvG.jpg">
  <br/>
  <i><a href=http://www.wildbunny.co.uk/blog/2012/10/09/how-to-make-a-multi-player-game-part-1/>Source: How to make a multiplayer game</a></i>
</p>

TCP is a connection-oriented protocol over an [IP network](https://en.wikipedia.org/wiki/Internet_Protocol).  Connection is established and terminated using a [handshake](https://en.wikipedia.org/wiki/Handshaking).  All packets sent are guaranteed to reach the destination in the original order and without corruption through:

* Sequence numbers and [checksum fields](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Checksum_computation) for each packet
* [Acknowledgement](https://en.wikipedia.org/wiki/Acknowledgement_(data_networks)) packets and automatic retransmission

If the sender does not receive a correct response, it will resend the packets.  If there are multiple timeouts, the connection is dropped.  TCP also implements [flow control](https://en.wikipedia.org/wiki/Flow_control_(data)) and [congestion control](https://en.wikipedia.org/wiki/Network_congestion#Congestion_control).  These guarantees cause delays and generally result in less efficient transmission than UDP.

To ensure high throughput, web servers can keep a large number of TCP connections open, resulting in high memory usage.  It can be expensive to have a large number of open connections between web server threads and say, a [memcached](#memcached) server.  [Connection pooling](https://en.wikipedia.org/wiki/Connection_pool) can help in addition to switching to UDP where applicable.

TCP is useful for applications that require high reliability but are less time critical.  Some examples include web servers, database info, SMTP, FTP, and SSH.

Use TCP over UDP when:

* You need all of the data to arrive intact
* You want to automatically make a best estimate use of the network throughput

### User datagram protocol (UDP)

<p align="center">
  <img src="http://i.imgur.com/yzDrJtA.jpg">
  <br/>
  <i><a href=http://www.wildbunny.co.uk/blog/2012/10/09/how-to-make-a-multi-player-game-part-1/>Source: How to make a multiplayer game</a></i>
</p>

UDP is connectionless.  Datagrams (analogous to packets) are guaranteed only at the datagram level.  Datagrams might reach their destination out of order or not at all.  UDP does not support congestion control.  Without the guarantees that TCP support, UDP is generally more efficient.

UDP can broadcast, sending datagrams to all devices on the subnet.  This is useful with [DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol) because the client has not yet received an IP address, thus preventing a way for TCP to stream without the IP address.

UDP is less reliable but works well in real time use cases such as VoIP, video chat, streaming, and realtime multiplayer games.

Use UDP over TCP when:

* You need the lowest latency
* Late data is worse than loss of data
* You want to implement your own error correction

#### Source(s) and further reading: TCP and UDP

* [Networking for game programming](http://gafferongames.com/networking-for-game-programmers/udp-vs-tcp/)
* [Key differences between TCP and UDP protocols](http://www.cyberciti.biz/faq/key-differences-between-tcp-and-udp-protocols/)
* [Difference between TCP and UDP](http://stackoverflow.com/questions/5970383/difference-between-tcp-and-udp)
* [Transmission control protocol](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)
* [User datagram protocol](https://en.wikipedia.org/wiki/User_Datagram_Protocol)
* [Scaling memcache at Facebook](http://www.cs.bu.edu/~jappavoo/jappavoo.github.com/451/papers/memcache-fb.pdf)

### Remote procedure call (RPC)

<p align="center">
  <img src="http://i.imgur.com/iF4Mkb5.png">
  <br/>
  <i><a href=http://www.puncsky.com/blog/2016/02/14/crack-the-system-design-interview/>Source: Crack the system design interview</a></i>
</p>

In an RPC, a client causes a procedure to execute on a different address space, usually a remote server.  The procedure is coded as if it were a local procedure call, abstracting away the details of how to communicate with the server from the client program.  Remote calls are usually slower and less reliable than local calls so it is helpful to distinguish RPC calls from local calls.  Popular RPC frameworks include [Protobuf](https://developers.google.com/protocol-buffers/), [Thrift](https://thrift.apache.org/), and [Avro](https://avro.apache.org/docs/current/).

RPC is a request-response protocol:

* **Client program** - Calls the client stub procedure.  The parameters are pushed onto the stack like a local procedure call.
* **Client stub procedure** - Marshals (packs) procedure id and arguments into a request message.
* **Client communication module** - OS sends the message from the client to the server.
* **Server communication module** - OS passes the incoming packets to the server stub procedure.
* **Server stub procedure** -  Unmarshalls the results, calls the server procedure matching the procedure id and passes the given arguments.
* The server response repeats the steps above in reverse order.

Sample RPC calls:

```
GET /someoperation?data=anId

POST /anotheroperation
{
  "data":"anId";
  "anotherdata": "another value"
}
```

RPC is focused on exposing behaviors.  RPCs are often used for performance reasons with internal communications, as you can hand-craft native calls to better fit your use cases.

Choose a native library (aka SDK) when:

* You know your target platform.
* You want to control how your "logic" is accessed.
* You want to control how error control happens off your library.
* Performance and end user experience is your primary concern.

HTTP APIs following **REST** tend to be used more often for public APIs.

#### Disadvantage(s): RPC

* RPC clients become tightly coupled to the service implementation.
* A new API must be defined for every new operation or use case.
* It can be difficult to debug RPC.
* You might not be able to leverage existing technologies out of the box.  For example, it might require additional effort to ensure [RPC calls are properly cached](http://etherealbits.com/2012/12/debunking-the-myths-of-rpc-rest/) on caching servers such as [Squid](http://www.squid-cache.org/).

### Representational state transfer (REST)

REST is an architectural style enforcing a client/server model where the client acts on a set of resources managed by the server.  The server provides a representation of resources and actions that can either manipulate or get a new representation of resources.  All communication must be stateless and cacheable.

There are four qualities of a RESTful interface:

* **Identify resources (URI in HTTP)** - use the same URI regardless of any operation.
* **Change with representations (Verbs in HTTP)** - use verbs, headers, and body.
* **Self-descriptive error message (status response in HTTP)** - Use status codes, don't reinvent the wheel.
* **[HATEOAS](http://restcookbook.com/Basics/hateoas/) (HTML interface for HTTP)** - your web service should be fully accessible in a browser.

Sample REST calls:

```
GET /someresources/anId

PUT /someresources/anId
{"anotherdata": "another value"}
```

REST is focused on exposing data.  It minimizes the coupling between client/server and is often used for public HTTP APIs.  REST uses a more generic and uniform method of exposing resources through URIs, [representation through headers](https://github.com/for-GET/know-your-http-well/blob/master/headers.md), and actions through verbs such as GET, POST, PUT, DELETE, and PATCH.  Being stateless, REST is great for horizontal scaling and partitioning.

#### Disadvantage(s): REST

* With REST being focused on exposing data, it might not be a good fit if resources are not naturally organized or accessed in a simple hierarchy.  For example, returning all updated records from the past hour matching a particular set of events is not easily expressed as a path.  With REST, it is likely to be implemented with a combination of URI path, query parameters, and possibly the request body.
* REST typically relies on a few verbs (GET, POST, PUT, DELETE, and PATCH) which sometimes doesn't fit your use case.  For example, moving expired documents to the archive folder might not cleanly fit within these verbs.
* Fetching complicated resources with nested hierarchies requires multiple round trips between the client and server to render single views, e.g. fetching content of a blog entry and the comments on that entry. For mobile applications operating in variable network conditions, these multiple roundtrips are highly undesirable.
* Over time, more fields might be added to an API response and older clients will receive all new data fields, even those that they do not need, as a result, it bloats the payload size and leads to larger latencies.

### RPC and REST calls comparison

| Operation | RPC | REST |
|---|---|---|
| Signup	| **POST** /signup | **POST** /persons |
| Resign	| **POST** /resign<br/>{<br/>"personid": "1234"<br/>} | **DELETE** /persons/1234 |
| Read a person | **GET** /readPerson?personid=1234 | **GET** /persons/1234 |
| Read a person’s items list | **GET** /readUsersItemsList?personid=1234 | **GET** /persons/1234/items |
| Add an item to a person’s items | **POST** /addItemToUsersItemsList<br/>{<br/>"personid": "1234";<br/>"itemid": "456"<br/>} | **POST** /persons/1234/items<br/>{<br/>"itemid": "456"<br/>} |
| Update an item	| **POST** /modifyItem<br/>{<br/>"itemid": "456";<br/>"key": "value"<br/>} | **PUT** /items/456<br/>{<br/>"key": "value"<br/>} |
| Delete an item | **POST** /removeItem<br/>{<br/>"itemid": "456"<br/>} | **DELETE** /items/456 |

<p align="center">
  <i><a href=https://apihandyman.io/do-you-really-know-why-you-prefer-rest-over-rpc/>Source: Do you really know why you prefer REST over RPC</a></i>
</p>

#### Source(s) and further reading: REST and RPC

* [Do you really know why you prefer REST over RPC](https://apihandyman.io/do-you-really-know-why-you-prefer-rest-over-rpc/)
* [When are RPC-ish approaches more appropriate than REST?](http://programmers.stackexchange.com/a/181186)
* [REST vs JSON-RPC](http://stackoverflow.com/questions/15056878/rest-vs-json-rpc)
* [Debunking the myths of RPC and REST](http://etherealbits.com/2012/12/debunking-the-myths-of-rpc-rest/)
* [What are the drawbacks of using REST](https://www.quora.com/What-are-the-drawbacks-of-using-RESTful-APIs)
* [Crack the system design interview](http://www.puncsky.com/blog/2016/02/14/crack-the-system-design-interview/)
* [Thrift](https://code.facebook.com/posts/1468950976659943/)
* [Why REST for internal use and not RPC](http://arstechnica.com/civis/viewtopic.php?t=1190508)

## Security

This section could use some updates.  Consider [contributing](#contributing)!

Security is a broad topic.  Unless you have considerable experience, a security background, or are applying for a position that requires knowledge of security, you probably won't need to know more than the basics:

* Encrypt in transit and at rest.
* Sanitize all user inputs or any input parameters exposed to user to prevent [XSS](https://en.wikipedia.org/wiki/Cross-site_scripting) and [SQL injection](https://en.wikipedia.org/wiki/SQL_injection).
* Use parameterized queries to prevent SQL injection.
* Use the principle of [least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege).

### Source(s) and further reading

* [Security guide for developers](https://github.com/FallibleInc/security-guide-for-developers)
* [OWASP top ten](https://www.owasp.org/index.php/OWASP_Top_Ten_Cheat_Sheet)

## Appendix

You'll sometimes be asked to do 'back-of-the-envelope' estimates.  For example, you might need to determine how long it will take to generate 100 image thumbnails from disk or how much memory a data structure will take.  The **Powers of two table** and **Latency numbers every programmer should know** are handy references.

### Powers of two table

```
Power           Exact Value         Approx Value        Bytes
---------------------------------------------------------------
7                             128
8                             256
10                           1024   1 thousand           1 KB
16                         65,536                       64 KB
20                      1,048,576   1 million            1 MB
30                  1,073,741,824   1 billion            1 GB
32                  4,294,967,296                        4 GB
40              1,099,511,627,776   1 trillion           1 TB
```

#### Source(s) and further reading

* [Powers of two](https://en.wikipedia.org/wiki/Power_of_two)

### Latency numbers every programmer should know

```
Latency Comparison Numbers
--------------------------
L1 cache reference                           0.5 ns
Branch mispredict                            5   ns
L2 cache reference                           7   ns                      14x L1 cache
Mutex lock/unlock                          100   ns
Main memory reference                      100   ns                      20x L2 cache, 200x L1 cache
Compress 1K bytes with Zippy            10,000   ns       10 us
Send 1 KB bytes over 1 Gbps network     10,000   ns       10 us
Read 4 KB randomly from SSD*           150,000   ns      150 us          ~1GB/sec SSD
Read 1 MB sequentially from memory     250,000   ns      250 us
Round trip within same datacenter      500,000   ns      500 us
Read 1 MB sequentially from SSD*     1,000,000   ns    1,000 us    1 ms  ~1GB/sec SSD, 4X memory
Disk seek                           10,000,000   ns   10,000 us   10 ms  20x datacenter roundtrip
Read 1 MB sequentially from 1 Gbps  10,000,000   ns   10,000 us   10 ms  40x memory, 10X SSD
Read 1 MB sequentially from disk    30,000,000   ns   30,000 us   30 ms 120x memory, 30X SSD
Send packet CA->Netherlands->CA    150,000,000   ns  150,000 us  150 ms

Notes
-----
1 ns = 10^-9 seconds
1 us = 10^-6 seconds = 1,000 ns
1 ms = 10^-3 seconds = 1,000 us = 1,000,000 ns
```

Handy metrics based on numbers above:

* Read sequentially from disk at 30 MB/s
* Read sequentially from 1 Gbps Ethernet at 100 MB/s
* Read sequentially from SSD at 1 GB/s
* Read sequentially from main memory at 4 GB/s
* 6-7 world-wide round trips per second
* 2,000 round trips per second within a data center

#### Latency numbers visualized

![](https://camo.githubusercontent.com/77f72259e1eb58596b564d1ad823af1853bc60a3/687474703a2f2f692e696d6775722e636f6d2f6b307431652e706e67)

#### Source(s) and further reading

* [Latency numbers every programmer should know - 1](https://gist.github.com/jboner/2841832)
* [Latency numbers every programmer should know - 2](https://gist.github.com/hellerbarde/2843375)
* [Designs, lessons, and advice from building large distributed systems](http://www.cs.cornell.edu/projects/ladis2009/talks/dean-keynote-ladis2009.pdf)
* [Software Engineering Advice from Building Large-Scale Distributed Systems](https://static.googleusercontent.com/media/research.google.com/en//people/jeff/stanford-295-talk.pdf)

### Additional system design interview questions

> Common system design interview questions, with links to resources on how to solve each.

| Question | Reference(s) |
|---|---|
| Design a file sync service like Dropbox | [youtube.com](https://www.youtube.com/watch?v=PE4gwstWhmc) |
| Design a search engine like Google | [queue.acm.org](http://queue.acm.org/detail.cfm?id=988407)<br/>[stackexchange.com](http://programmers.stackexchange.com/questions/38324/interview-question-how-would-you-implement-google-search)<br/>[ardendertat.com](http://www.ardendertat.com/2012/01/11/implementing-search-engines/)<br>[stanford.edu](http://infolab.stanford.edu/~backrub/google.html) |
| Design a scalable web crawler like Google | [quora.com](https://www.quora.com/How-can-I-build-a-web-crawler-from-scratch) |
| Design Google docs | [code.google.com](https://code.google.com/p/google-mobwrite/)<br/>[neil.fraser.name](https://neil.fraser.name/writing/sync/) |
| Design a key-value store like Redis | [slideshare.net](http://www.slideshare.net/dvirsky/introduction-to-redis) |
| Design a cache system like Memcached | [slideshare.net](http://www.slideshare.net/oemebamo/introduction-to-memcached) |
| Design a recommendation system like Amazon's | [hulu.com](http://tech.hulu.com/blog/2011/09/19/recommendation-system.html)<br/>[ijcai13.org](http://ijcai13.org/files/tutorial_slides/td3.pdf) |
| Design a tinyurl system like Bitly | [n00tc0d3r.blogspot.com](http://n00tc0d3r.blogspot.com/) |
| Design a chat app like WhatsApp | [highscalability.com](http://highscalability.com/blog/2014/2/26/the-whatsapp-architecture-facebook-bought-for-19-billion.html)
| Design a picture sharing system like Instagram | [highscalability.com](http://highscalability.com/flickr-architecture)<br/>[highscalability.com](http://highscalability.com/blog/2011/12/6/instagram-architecture-14-million-users-terabytes-of-photos.html) |
| Design the Facebook news feed function | [quora.com](http://www.quora.com/What-are-best-practices-for-building-something-like-a-News-Feed)<br/>[quora.com](http://www.quora.com/Activity-Streams/What-are-the-scaling-issues-to-keep-in-mind-while-developing-a-social-network-feed)<br/>[slideshare.net](http://www.slideshare.net/danmckinley/etsy-activity-feeds-architecture) |
| Design the Facebook timeline function | [facebook.com](https://www.facebook.com/note.php?note_id=10150468255628920)<br/>[highscalability.com](http://highscalability.com/blog/2012/1/23/facebook-timeline-brought-to-you-by-the-power-of-denormaliza.html) |
| Design the Facebook chat function | [erlang-factory.com](http://www.erlang-factory.com/upload/presentations/31/EugeneLetuchy-ErlangatFacebook.pdf)<br/>[facebook.com](https://www.facebook.com/note.php?note_id=14218138919&id=9445547199&index=0) |
| Design a graph search function like Facebook's | [facebook.com](https://www.facebook.com/notes/facebook-engineering/under-the-hood-building-out-the-infrastructure-for-graph-search/10151347573598920)<br/>[facebook.com](https://www.facebook.com/notes/facebook-engineering/under-the-hood-indexing-and-ranking-in-graph-search/10151361720763920)<br/>[facebook.com](https://www.facebook.com/notes/facebook-engineering/under-the-hood-the-natural-language-interface-of-graph-search/10151432733048920) |
| Design a content delivery network like CloudFlare | [cmu.edu](http://repository.cmu.edu/cgi/viewcontent.cgi?article=2112&context=compsci) |
| Design a trending topic system like Twitter's | [michael-noll.com](http://www.michael-noll.com/blog/2013/01/18/implementing-real-time-trending-topics-in-storm/)<br/>[snikolov .wordpress.com](http://snikolov.wordpress.com/2012/11/14/early-detection-of-twitter-trends/) |
| Design a random ID generation system | [blog.twitter.com](https://blog.twitter.com/2010/announcing-snowflake)<br/>[github.com](https://github.com/twitter/snowflake/) |
| Return the top k requests during a time interval | [ucsb.edu](https://icmi.cs.ucsb.edu/research/tech_reports/reports/2005-23.pdf)<br/>[wpi.edu](http://davis.wpi.edu/xmdv/docs/EDBT11-diyang.pdf) |
| Design a system that serves data from multiple data centers | [highscalability.com](http://highscalability.com/blog/2009/8/24/how-google-serves-data-from-multiple-datacenters.html) |
| Design an online multiplayer card game | [indieflashblog.com](http://www.indieflashblog.com/how-to-create-an-asynchronous-multiplayer-game.html)<br/>[buildnewgames.com](http://buildnewgames.com/real-time-multiplayer/) |
| Design a garbage collection system | [stuffwithstuff.com](http://journal.stuffwithstuff.com/2013/12/08/babys-first-garbage-collector/)<br/>[washington.edu](http://courses.cs.washington.edu/courses/csep521/07wi/prj/rick.pdf) |
| Add a system design question | [Contribute](#contributing) |

### Real world architectures

> Articles on how real world systems are designed.

<p align="center">
  <img src="http://i.imgur.com/TcUo2fw.png">
  <br/>
  <i><a href=https://www.infoq.com/presentations/Twitter-Timeline-Scalability>Source: Twitter timelines at scale</a></i>
</p>

**Don't focus on nitty gritty details for the following articles, instead:**

* Identify shared principles, common technologies, and patterns within these articles
* Study what problems are solved by each component, where it works, where it doesn't
* Review the lessons learned

|Type | System | Reference(s) |
|---|---|---|
| Data processing | **MapReduce** - Distributed data processing from Google | [research.google.com](http://static.googleusercontent.com/media/research.google.com/zh-CN/us/archive/mapreduce-osdi04.pdf) |
| Data processing | **Spark** - Distributed data processing from Databricks | [slideshare.net](http://www.slideshare.net/AGrishchenko/apache-spark-architecture) |
| Data processing | **Storm** - Distributed data processing from Twitter | [slideshare.net](http://www.slideshare.net/previa/storm-16094009) |
| | | |
| Data store | **Bigtable** - Distributed column-oriented database from Google | [harvard.edu](http://www.read.seas.harvard.edu/~kohler/class/cs239-w08/chang06bigtable.pdf) |
| Data store | **HBase** - Open source implementation of Bigtable | [slideshare.net](http://www.slideshare.net/alexbaranau/intro-to-hbase) |
| Data store | **Cassandra** - Distributed column-oriented database from Facebook | [slideshare.net](http://www.slideshare.net/planetcassandra/cassandra-introduction-features-30103666)
| Data store | **DynamoDB** - Document-oriented database from Amazon | [harvard.edu](http://www.read.seas.harvard.edu/~kohler/class/cs239-w08/decandia07dynamo.pdf) |
| Data store | **MongoDB** - Document-oriented database | [slideshare.net](http://www.slideshare.net/mdirolf/introduction-to-mongodb) |
| Data store | **Spanner** - Globally-distributed database from Google | [research.google.com](http://research.google.com/archive/spanner-osdi2012.pdf) |
| Data store | **Memcached** - Distributed memory caching system | [slideshare.net](http://www.slideshare.net/oemebamo/introduction-to-memcached) |
| Data store | **Redis** - Distributed memory caching system with persistence and value types | [slideshare.net](http://www.slideshare.net/dvirsky/introduction-to-redis) |
| | | |
| File system | **Google File System (GFS)** - Distributed file system | [research.google.com](http://static.googleusercontent.com/media/research.google.com/zh-CN/us/archive/gfs-sosp2003.pdf) |
| File system | **Hadoop File System (HDFS)** - Open source implementation of GFS | [apache.org](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html) |
| | | |
| Misc | **Chubby** - Lock service for loosely-coupled distributed systems from Google | [research.google.com](http://static.googleusercontent.com/external_content/untrusted_dlcp/research.google.com/en/us/archive/chubby-osdi06.pdf) |
| Misc | **Dapper** - Distributed systems tracing infrastructure | [research.google.com](http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/36356.pdf)
| Misc | **Kafka** - Pub/sub message queue from LinkedIn | [slideshare.net](http://www.slideshare.net/mumrah/kafka-talk-tri-hug) |
| Misc | **Zookeeper** - Centralized infrastructure and services enabling synchronization | [slideshare.net](http://www.slideshare.net/sauravhaloi/introduction-to-apache-zookeeper) |
| | Add an architecture | [Contribute](#contributing) |

### Company architectures

| Company | Reference(s) |
|---|---|
| Amazon | [Amazon architecture](http://highscalability.com/amazon-architecture) |
| Cinchcast | [Producing 1,500 hours of audio every day](http://highscalability.com/blog/2012/7/16/cinchcast-architecture-producing-1500-hours-of-audio-every-d.html) |
| DataSift | [Realtime datamining At 120,000 tweets per second](http://highscalability.com/blog/2011/11/29/datasift-architecture-realtime-datamining-at-120000-tweets-p.html) |
| DropBox | [How we've scaled Dropbox](https://www.youtube.com/watch?v=PE4gwstWhmc) |
| ESPN | [Operating At 100,000 duh nuh nuhs per second](http://highscalability.com/blog/2013/11/4/espns-architecture-at-scale-operating-at-100000-duh-nuh-nuhs.html) |
| Google | [Google architecture](http://highscalability.com/google-architecture) |
| Instagram | [14 million users, terabytes of photos](http://highscalability.com/blog/2011/12/6/instagram-architecture-14-million-users-terabytes-of-photos.html)<br/>[What powers Instagram](http://instagram-engineering.tumblr.com/post/13649370142/what-powers-instagram-hundreds-of-instances) |
| Justin.tv | [Justin.Tv's live video broadcasting architecture](http://highscalability.com/blog/2010/3/16/justintvs-live-video-broadcasting-architecture.html) |
| Facebook | [Scaling memcached at Facebook](https://cs.uwaterloo.ca/~brecht/courses/854-Emerging-2014/readings/key-value/fb-memcached-nsdi-2013.pdf)<br/>[TAO: Facebook’s distributed data store for the social graph](https://cs.uwaterloo.ca/~brecht/courses/854-Emerging-2014/readings/data-store/tao-facebook-distributed-datastore-atc-2013.pdf)<br/>[Facebook’s photo storage](https://www.usenix.org/legacy/event/osdi10/tech/full_papers/Beaver.pdf) |
| Flickr | [Flickr architecture](http://highscalability.com/flickr-architecture) |
| Mailbox | [From 0 to one million users in 6 weeks](http://highscalability.com/blog/2013/6/18/scaling-mailbox-from-0-to-one-million-users-in-6-weeks-and-1.html) |
| Pinterest | [From 0 To 10s of billions of page views a month](http://highscalability.com/blog/2013/4/15/scaling-pinterest-from-0-to-10s-of-billions-of-page-views-a.html)<br/>[18 million visitors, 10x growth, 12 employees](http://highscalability.com/blog/2012/5/21/pinterest-architecture-update-18-million-visitors-10x-growth.html) |
| Playfish | [50 million monthly users and growing](http://highscalability.com/blog/2010/9/21/playfishs-social-gaming-architecture-50-million-monthly-user.html) |
| PlentyOfFish | [PlentyOfFish architecture](http://highscalability.com/plentyoffish-architecture) |
| Salesforce | [How they handle 1.3 billion transactions a day](http://highscalability.com/blog/2013/9/23/salesforce-architecture-how-they-handle-13-billion-transacti.html) |
| Stack Overflow | [Stack Overflow architecture](http://highscalability.com/blog/2009/8/5/stack-overflow-architecture.html) |
| TripAdvisor | [40M visitors, 200M dynamic page views, 30TB data](http://highscalability.com/blog/2011/6/27/tripadvisor-architecture-40m-visitors-200m-dynamic-page-view.html) |
| Tumblr | [15 billion page views a month](http://highscalability.com/blog/2012/2/13/tumblr-architecture-15-billion-page-views-a-month-and-harder.html) |
| Twitter | [Making Twitter 10000 percent faster](http://highscalability.com/scaling-twitter-making-twitter-10000-percent-faster)<br/>[Storing 250 million tweets a day using MySQL](http://highscalability.com/blog/2011/12/19/how-twitter-stores-250-million-tweets-a-day-using-mysql.html)<br/>[150M active users, 300K QPS, a 22 MB/S firehose](http://highscalability.com/blog/2013/7/8/the-architecture-twitter-uses-to-deal-with-150m-active-users.html)<br/>[Timelines at scale](https://www.infoq.com/presentations/Twitter-Timeline-Scalability)<br/>[Big and small data at Twitter](https://www.youtube.com/watch?v=5cKTP36HVgI)<br/>[Operations at Twitter: scaling beyond 100 million users](https://www.youtube.com/watch?v=z8LU0Cj6BOU) |
| Uber | [How Uber scales their real-time market platform](http://highscalability.com/blog/2015/9/14/how-uber-scales-their-real-time-market-platform.html) |
| WhatsApp | [The WhatsApp architecture Facebook bought for $19 billion](http://highscalability.com/blog/2014/2/26/the-whatsapp-architecture-facebook-bought-for-19-billion.html) |
| YouTube | [YouTube scalability](https://www.youtube.com/watch?v=w5WVu624fY8)<br/>[YouTube architecture](http://highscalability.com/youtube-architecture) |

### Company engineering blogs

> Architectures for companies you are interviewing with.
>
> Questions you encounter might be from the same domain.

* [Airbnb Engineering](http://nerds.airbnb.com/)
* [Atlassian Developers](https://developer.atlassian.com/blog/)
* [Autodesk Engineering](http://cloudengineering.autodesk.com/blog/)
* [AWS Blog](https://aws.amazon.com/blogs/aws/)
* [Bitly Engineering Blog](http://word.bitly.com/)
* [Box Blogs](https://www.box.com/blog/engineering/)
* [Cloudera Developer Blog](http://blog.cloudera.com/blog/)
* [Dropbox Tech Blog](https://tech.dropbox.com/)
* [Engineering at Quora](http://engineering.quora.com/)
* [Ebay Tech Blog](http://www.ebaytechblog.com/)
* [Evernote Tech Blog](https://blog.evernote.com/tech/)
* [Etsy Code as Craft](http://codeascraft.com/)
* [Facebook Engineering](https://www.facebook.com/Engineering)
* [Flickr Code](http://code.flickr.net/)
* [Foursquare Engineering Blog](http://engineering.foursquare.com/)
* [GitHub Engineering Blog](http://githubengineering.com/)
* [Google Research Blog](http://googleresearch.blogspot.com/)
* [Groupon Engineering Blog](https://engineering.groupon.com/)
* [Heroku Engineering Blog](https://engineering.heroku.com/)
* [Hubspot Engineering Blog](http://product.hubspot.com/blog/topic/engineering)
* [High Scalability](http://highscalability.com/)
* [Instagram Engineering](http://instagram-engineering.tumblr.com/)
* [Intel Software Blog](https://software.intel.com/en-us/blogs/)
* [Jane Street Tech Blog](https://blogs.janestreet.com/category/ocaml/)
* [LinkedIn Engineering](http://engineering.linkedin.com/blog)
* [Microsoft Engineering](https://engineering.microsoft.com/)
* [Microsoft Python Engineering](https://blogs.msdn.microsoft.com/pythonengineering/)
* [Netflix Tech Blog](http://techblog.netflix.com/)
* [Paypal Developer Blog](https://devblog.paypal.com/category/engineering/)
* [Pinterest Engineering Blog](http://engineering.pinterest.com/)
* [Quora Engineering](https://engineering.quora.com/)
* [Reddit Blog](http://www.redditblog.com/)
* [Salesforce Engineering Blog](https://developer.salesforce.com/blogs/engineering/)
* [Slack Engineering Blog](https://slack.engineering/)
* [Spotify Labs](https://labs.spotify.com/)
* [Twilio Engineering Blog](http://www.twilio.com/engineering)
* [Twitter Engineering](https://engineering.twitter.com/)
* [Uber Engineering Blog](http://eng.uber.com/)
* [Yahoo Engineering Blog](http://yahooeng.tumblr.com/)
* [Yelp Engineering Blog](http://engineeringblog.yelp.com/)
* [Zynga Engineering Blog](https://www.zynga.com/blogs/engineering)

#### Source(s) and further reading

* [kilimchoi/engineering-blogs](https://github.com/kilimchoi/engineering-blogs)

## Under development

Interested in adding a section or helping complete one in-progress?  [Contribute](#contributing)!

* Distributed computing with MapReduce
* Consistent hashing
* Scatter gather
* [Contribute](#contributing)

## Credits

Credits and sources are provided throughout this repo.

Special thanks to:

* [Hired in tech](http://www.hiredintech.com/system-design/the-system-design-process/)
* [Cracking the coding interview](https://www.amazon.com/dp/0984782850/)
* [High scalability](http://highscalability.com/)
* [checkcheckzz/system-design-interview](https://github.com/checkcheckzz/system-design-interview)
* [shashank88/system_design](https://github.com/shashank88/system_design)
* [mmcgrana/services-engineering](https://github.com/mmcgrana/services-engineering)
* [System design cheat sheet](https://gist.github.com/vasanthk/485d1c25737e8e72759f)
* [A distributed systems reading list](http://dancres.github.io/Pages/)
* [Cracking the system design interview](http://www.puncsky.com/blog/2016/02/14/crack-the-system-design-interview/)

## Contact info

Feel free to contact me to discuss any issues, questions, or comments.

My contact info can be found on my [GitHub page](https://github.com/donnemartin).

## License

*I am providing code and resources in this repository to you under an open source license.  Because this is my personal repository, the license you receive to my code and resources is from me and not my employer (Facebook).*

    Copyright 2017 Donne Martin

    Creative Commons Attribution 4.0 International License (CC BY 4.0)

    http://creativecommons.org/licenses/by/4.0/
