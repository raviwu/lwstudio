---
title: "Rails grape API 上實作 JWT 多重登入"
date: 2016-06-10T05:05:44+08:00
tags: ["rails", "jwt", "coding"]
disqus: false
draft: false
---

多重登入的需求，在服務本身允許「跨裝置體驗」或者是「接受不同 request agent (APP request / mobile browser)」的時候顯得重要。

理想情況下，可以讓使用者在介面上面清楚知道目前帳號的登入狀態，並在登入數超過服務上限的時候，讓使用者自己選擇要移除登入授權的裝置，使用者流程大概會像（[這篇文](https://www.authy.com/blog/multi-multi-factor-authentication)）那樣，要做到類似參考連結這樣的管理介面，需要<strike>我目前還不會的</strike>前端的配合建置與基本的 session 管理 API。本篇只先建置允許使用多重登入的資料結構與登入驗證 API。

選擇用 [JWT (Jason Web Token)](https://jwt.io/) 作為主要的憑證溝通，優點有一些：

1.  透過加密 payload 來比對資料庫內容的方式，可以不用在資料表中「直接存入」與憑證相關的訊息，加密跟解密只要透過協定就能進行調整。
2.  payload 裡面塞入的訊息可以自訂，在各服務跟裝置之間交換加密過後的使用者資料。
3.  signing key / protocol 可以在有資安疑慮時更換。

使用 Devise 的 Rails 捧油也可參考 [devise_token_auth](https://github.com/lynndylanhurley/devise_token_auth) 這個 gem，整合效果在文件上面看起來很完善，但因為目前我手邊的系統絕大多數的 request 都來自 grape API endpoint ，比較過後，跟搞懂並駕馭 devise 設定比起來，我還是選擇自刻 JWT 較節省開發時間。

* * *

在 Rails grape API 上實作 JWT 多重登入，分 4 個階段。

1.  建立 user_sessions 資料表跟 UserSession Model Setup，目前找得到的 [Rails JWT 整合教學](https://www.sitepoint.com/introduction-to-using-jwt-in-rails/)大多是用在 Controller / Rails View 上，為了之後如果要導入前端管理裝置時可以很方便地去刪除跟紀錄不要的 Session，我選擇了透過 UserSession 這個 model 來做媒介，之後要新增 API 的時候直接去對這個 table 做 query 跟各種欄位處理，盡量把 Session 管理跟目前龐大的 users table 切乾淨。
2.  寫一隻自己的 JasonWebToken Service 把 jwt 的 code 包裝得更適合自己的系統使用：雖然 jwt 這隻 gem 的語法已經寫得很美，但還想更方便地只要把 headers 塞進去這個 wrapper 就可以得到 user object 回傳或者是指定的錯誤訊息。
3.  做好資料結構跟 JasonWebToken Service 後，就可以開始使用準備好的工具來更換 Grape API 的登入跟檢查憑證機制，首先要在登入的 API output 提供 jwt 字串。
4.  最後，在檢查憑證機制的 helper 裡使用 JasonWebToken class 去 decode request 裡面的 jwt。
