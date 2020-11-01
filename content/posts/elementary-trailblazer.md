+++
title = "初探 Trailblazer 框架"
description = ""
date = 2017-01-07T13:23:29+08:00
tags = ["coding", "rails", "ruby", "trailblazer"]
draft = false
+++

最近公司的 Rails 專案試用了 [Trailblazer](http://trailblazer.to/) 這套整理 Rails 程式碼的框架。（目前使用的是 1.1 版）

Trailblazer 是擺在 Rails 上的一套工具，雖然有本[專書](https://leanpub.com/trailblazer)可以翻找，但實際上就是個自成一格的整理程式碼套路，除了要學新的 DSL 外，在 API 文件沒有寫得非常詳細，加上使用者也沒多到可以 stackoverflow 的情況下，假如開發需求和設計者 [Nick Sutterer](https://github.com/apotonick) 設想的情況不一樣時，小小撞牆是難免的。

Trailblazer 是模組化設計，所以並非所有模組需要安裝才能開始享受 Trailblazer 帶來的方便感，除了主要的 Operation / Policy / Contract / Cell 等稍微有摸過外，其他的眉眉角角尚待開發。稍微整理一下截至目前為止使用這套框架的個人認知，這套 Trailblazer 的目的是將原先 Rails 單純 MVC 架構下可能會散亂在 Controller / Model 層的商業邏輯，集中起來包在 [Operation](http://trailblazer.to/gems/operation/1.1/index.html) 的概念裡，（在理想狀況下）分別簡化日益肥大的 Controller / Model / View 邏輯：

*   Controller
    *   讓 controller 單純負責傳送 current_user / resource 到正確的 operation 並視操作結果導向相對應的路由
    *   權限控管交給 [Policy](http://trailblazer.to/gems/operation/1.1/policy.html) 處理，用 controller 塞給 operation 的 `current_user` 配合 operation 的 `model!(param)` method 所傳回之 resource 兩者之間的關係去做權限控管
    *   商業邏輯封裝在 Operation 裡
    *   只傳送成功執行 operation 後的 `@model` (resource) 給 view
*   Model
    *   讓 Model 單純負責與資料庫之間的溝通，原先的 Validation 改由 Contract 操作
    *   本來可能塞在 Model 裡的各種 `before_validation` `before_save` 等 callbacks 拉出來改在 operation 的 process 裡面操作
    *   本來塞在 “fat model” 裡的商業邏輯，一樣抽出來寫在相對應的 operation 裡
*   View
    *   減少原先直接吃 controller instance variables 並在 erb 或 global helper 裡面塞各種邏輯判斷的情況，把 controller 傳過來的 `@model` 傳到相對應的 Cell 裡面去處理。
    *   Cell 結合 Reform 可以做出複雜的 form 格式，由於已經透過抽象化，所以複雜 form 的參數不再直接對應到 Model 上，而是綁在 contract attribute 上，屬性的自由度可以大幅增加，透過 [prepopulator](http://trailblazer.to/gems/reform/prepopulator.html) 去準備需要呈現的 `@contract`，餵進 controller 的參數也可以透過 [populator](http://trailblazer.to/gems/reform/populator.html) 做處理，讓 Cell 可以用 form_for 的方式去打包 contract 以簡化 view code

其他尚未摸到邊但好像不錯用的模組是 [Representable](http://trailblazer.to/gems/representable/) 跟 [Roar](http://trailblazer.to/gems/roar/jsonapi.html) ，主要用在做 API 上，有心得再補上。