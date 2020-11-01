+++
title = "在 rails 上實作轉移 Parse 手機推播服務到 Amazon SNS"
description = ""
date = 2016-06-24T05:05:44+08:00
tags = ["rails", "parse", "aws", "sns", "push notification", "coding"]
draft = false
+++

### 簡要說明

為了避免使用者因服務轉換而被迫強制更新，移轉過程採用漸進方式。

如果不需要如此痛苦的同時使用兩種服務的捧油，可以參考[這一篇](https://mobile.awsblog.com/post/Tx3NE69QDHI7LJK/Migrating-from-Parse-Push-to-Amazon-SNS)的實作，直接用 AWS Mobile Hub 的服務做批次資料移轉。

由於在轉換期需要同時支援兩邊服務運作，又要避免使用者重複收到相同的推播，需要實作「對同一裝置擇一服務進行推播」。本篇實作的概念是，使用者只有在用新版本的 APP 登入時，伺服器才會從 APP 取得 device_token 去註冊 Amazon SNS endpoint_arn，同時刪除 Parse 服務上的使用者的所有 Installation 紀錄。

Parse 跟 Amazon SNS 的 API 邏輯有些不同，主要差別在於，Parse 只需要在同一 request 裡就可以達到「推播給符合某條件的 channels 組合，並且可以指定其中的特定 users 組合」，但是 Amazon 的推播就是 topic_arn / target_arn 兩種，要收到 topic_arn 的話要先讓 endpoint_arn 訂閱特定的 topic_arn，而 target_arn 則是一個 request 只能推播給單一 target_arn (endpoint_arn)。

### 開發步驟

1.  打包一隻 AmazonSnsWrapper 把 [Amazon SDK](http://docs.aws.amazon.com/sdkforruby/api/Aws/SNS.html) 提供的 method 客製成符合需求的介面，專門負責跟 Amazon server 溝通。
2.  新增 AmazonSnsInfo Model 以記錄使用者的 SNS 訂閱狀況。
3.  新增 ParseInstallationWrapper 打包 Parse RESTful API 裡的 method，查詢使用者之前透過 APP 註冊的 Installations，並且刪除已經成功轉換到 Amazon SNS 的 Installation。
4.  新增 update_push_subscription / logout API 來針對 APP 傳送的 device_token 做各種 push subscription / unsubscription 的操作。
5.  新增 PushServicesWrapper 把所有 Push 行為的邏輯打包在一起方便做服務切換。

### 一、 AmazonSnsWrapper

首先，我們得先建好 Amazon SNS 服務中 endpoint_arn / topic_arn / subscription_arn 新增、查詢、訂閱與刪除等機制，我習慣把第三方 API 的互動包成一個 Ruby Class 方便測試跟管理，建一個 AmazonSnsWrapper 來打包 SDK 相關的程式碼。

### 二、 AmazonSnsInfo

接著，因為管理 AWS SNS 註冊等事宜需要在資料庫裡存有相關的 platform_arn / endpoint_arn，所以新增一個 Model 叫 AmazonSnsInfo 來記錄使用者相關的 SNS 資料。

### 三、 ParseInstallationWrapper

由於 Parse 的 Channel push 還是會把訊息推播到曾經註冊過的有效 device token 上，導致 user 可能被 channel push 轟炸，在必須同時使用兩個服務的情況下，目前我只想到可以盡可能清掉 user 註冊在 Parse 相關的 Installation。

### 四、 update_push_subscription / logout API

透過 APP 的操作，可以確定使用者的使用狀態，在 APP launch 後提供最新的 device_token 到 update_push_subscription 去更新 endpoint_arn 與訂閱狀況，並且去 Parse 資料庫中撈。有時候同一裝置不一定會上回傳送一樣的 device_token，有時可能是因為使用者刪除 APP 又重裝之類的原因，如果導致 device_token 失效的話，在 Amazon SNS Console 裡這筆 endpoint 會被標註 `enabled == false` ，所以即使不在 Amazon SNS 系統上面刪除註冊，裝置也不會收到通知。

另外也有可能同一裝置、同一 device_token 但是不同的使用者登入，在測試跟評估後，採取直接刪除與 device_token 相關連的所有紀錄，刪除 AmazonSnsInfo 紀錄前因為有個 `before_destroy` 的 callback 去反訂閱 topic 並刪除 endpoint_arn 等相關 Amazon SNS Console 紀錄，確保推播不會送錯使用者。但剛剛討論到的那些 device_token 孤兒們基本上就是會晾在 Amazon SNS Console 裡了，有想到其他清 Amazon SNS Console 無用資料方式的捧油，小的在此跪求與我聯絡，感謝感謝。

規格要求還是要可以針對個別 device 的 target_arn 做推播，所以在 logout 的時候只替 endpoint 取消 topic 的訂閱。

### 五、 PushServicesWrapper

實作擇一推播相同 message 到 Parse / Amazon SNS 的機制。
