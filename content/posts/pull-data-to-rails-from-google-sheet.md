+++
title = "使用 Google Sheets API 拉資料到 Rails APP 裡"
description = ""
date = 2016-07-09T05:17:26+08:00
tags = ["rails", "google sheet", "coding"]
draft = false
+++

Google Sheets 很好用，一些程式中需要跨部門討論的字串定義，又或者是需要讓不方便存取原始碼的同事也能清楚知道原始碼裡部分設定字串的時候，通常都可以利用 Google Sheet 來做討論平台，再以 Google Sheets 上面的文件為基礎，匯入主程式中做各種處理，確保主程式的內容與 Google Sheets 上的內容同步。

要匯入 Google Sheets 內容到程式裡最簡單的方式，是輸出 csv 後寫個小 script 去讀取 csv 資料，轉成 Array 後就能靈活使用。

但若是開發過程當中常常要不斷更新 Google Sheets 欄位後再手動匯出 csv 跑 script 去後處理，這種半自動的方式還是令人覺得太過麻煩。可以考慮進一步用 Google Sheets API 去拉特定 Google Sheets 工作表資料，把半自動的「到瀏覽器視窗匯出 csv > 儲存檔案到特定資料夾 > 執行 script 完成匯入」簡化為「執行 script 完成匯入」。

首先到 [Google API Console](https://console.developers.google.com/apis/api/sheets.googleapis.com/overview) 申請一個可用的 API 身份，因為這隻 API 只要存取特定 Google Sheets 的內容，與其他 Google User 沒有互動，所以選用 [Service Account](https://support.google.com/googleapi/answer/6158849?hl=zh-Hant#serviceaccounts) 的授權方式就可以。

如果你的 Google 帳號之前沒申請過 Google API 的話，要先「建立專案」。

建立好專案後，進入 Google Sheets API 的啟用首頁，網址長得像「 https://console.developers.google.com/apis/api/sheets.googleapis.com/overview?project=your-project-name 」，按「啟動」就可以。

啟動後到「憑證」頁籤去設定到時候要使用的憑證類型，網址長得像「 https://console.developers.google.com/apis/credentials/wizard?api=sheets.googleapis.com&project=your-project-name 」，直接選擇「服務帳戶」。

![](/images/pull-data-to-rails-from-google-sheet/step-1.png)

進入服務帳戶的畫面後，選擇「建立服務帳戶」。

![](/images/pull-data-to-rails-from-google-sheet/step-2.png)

就會跳出指定帳戶名稱的視窗，勾選提供一組新私密金鑰的選項，選用建議的 JSON 格式。注意一下「服務帳戶 ID 」長得像一個「 email account 」，把這個「 email account 」看成到時使用這張 JSON 憑證存取時所代表的身份，簡言之，要把欲分享之 Google Sheets 文件的共享權限開給這個 email 帳號，看是只要 view 還是需要開 edit 的權限，取決於使用需求。

![](/images/pull-data-to-rails-from-google-sheet/step-3.png)

點選建立後就會跳出引導視窗下載 JSON 檔，**<span style="color: red;">請把這份 JSON 檔設到 .gitignore 裡</span>**，以免不小心公開<strike>你的秘密</strike>。接著就可以從 Google Sheets API 拿這隻 JSON 憑證去存取開給這個「 Service Account 」權限的文件了。

* * *

在 Rails 裡先裝 Google 提供的 gem 後，就可以寫一隻 Class 來存取指定的 Google Sheets 工作表：

寫好 GoogleSheetCrawler 這個 Class 後，就可以在 Rails 程式裡面使用，`GoogleSheetCrawler.get_sheet_array_from_google_sheet` 會拿到 csv 結構的 Array 去做各種數值處理，我通常會放在 `app/services/google_sheet_crawler.rb` 裡，視需求再透過 `app/jobs/update_status_code_from_google_sheets_job.rb` 之類的方式從 Console 手動更新或是 Cronjob 排定工作的方式去做更新。

不是很難的事，但這類日常重複性操作，還是能少一個步驟是一個。（茶）
