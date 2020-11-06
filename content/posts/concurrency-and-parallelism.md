+++
title = "多執行緒（Concurrency）與平行處理（Parallelism）"
description = ""
date = 2020-11-06T15:26:38+08:00
images = []
tags = ["coding", "concurrency", "parallelism"]
categories = ["coding"]
draft = false
+++

## 進程（Process）

使用系統資源執行程式的實體

- 電腦中執行程式的實體
- 每個進程都互相獨立
- Process 是執行緒的容器
- Process 會佔用系統資源
- 多工作業系統中可以同時執行數個 Process，但單個 CPU 一次只能執行一個 Process

## 線程、執行緒（Thread）

- 同個 Process 裡有至少個 Thread
- 同個 Process 裡的 Thread 共享系統資源
- 在多執行緒（Multithreading）環境中若兩個以上的執行緒對同一個變數進行改動可能產生死結（Deadlock）

## 多執行緒（Concurrency）

在同一進程的多執行緒環境中，把工作拆成數個子集，利用不同的執行緒分別完成每個子集。

## 平行處理（Parallelism）

透過負載平衡機制把數個工作分配到不同的工作單元裡「同時」進行。

## 參考文章

- [Concurrency 與 Parallelism 的不同之處](https://medium.com/mr-efacani-teatime/concurrency%E8%88%87parallelism%E7%9A%84%E4%B8%8D%E5%90%8C%E4%B9%8B%E8%99%95-1b212a020e30)
- [Concurrency vs. Parallelism — A brief view](https://medium.com/@itIsMadhavan/concurrency-vs-parallelism-a-brief-review-b337c8dac350)
