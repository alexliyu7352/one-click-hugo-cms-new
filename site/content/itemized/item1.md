---
title: NoHttp庫
m_slug: nohttp lib
author: Alex
categories:
  - Android
type: itemized
date: '2017-06-22'
description: Android实现Http标准协议框架，支持缓存（提供五种缓存模式）、代理、重定向，底层可动态切换OkHttp、URLConnection
featured: ''
featuredalt: ''
featuredpath: ''
format: Android
link: 'https://github.com/yanzhenjie/NoHttp'
linktitle: NoHttp
---
**Android实现Http标准协议框架，支持缓存（提供五种缓存模式）、代理、重定向，底层可动态切换OkHttp、URLConnection**

- - -

主要原因是, 之前使用的android-async-http這個庫越發的老舊並且力不從心了.從功能上來說目前的APP中還是工作良好的.

但是最重要的問題就是, 它不支持http2, 並且總是有一些新版本設備的適配問題, 比如說證書的驗證問題. 如果客戶端的系統的時間比證書籤發時間還要早, 那麼就會一直報錯. 哪怕你修改過系統時間了, 除非重啓否則還是會一直報錯.

類似的零陵散散的bug其實不少, 大部分的都已經陸續的通過修改源碼解決了.也就是因爲如此,所以一直都很難做決定遷移到新的庫上, 畢竟遷移成本會比較高, 而且遷移後不可避免的會需要一段時間解決新的問題.

但是目前來說仍舊需要遷移.對比了使用okhttp的幾個庫, 發現還是國人寫的這個Nohttp的功能比較適合. 因爲他封裝了大量的需要自己封裝的功能, 並且也適合多種場景.

- - -

目前只是打算使用這個庫, 還沒有正式進行遷移, 僅僅作爲記錄.稍後遷移時會做詳細的遷移記錄.
