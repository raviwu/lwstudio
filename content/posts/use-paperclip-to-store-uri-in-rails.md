---
title: "Rails 中使用 Paperclip 存 URI 附件"
date: 2016-05-12T04:38:49+08:00
tags: ["coding","rails","paperclip"]
disqus: false
draft: false
---

使用 Rails APP + Paperclip Gem 做附件存取系統，靜態檔案的資料夾格式預設是跟著 id 跑（ /000/000/000/ 九碼化的 ID 切成 3 階資料夾），最近因為要做資料庫合併，大批的資料會被賦予新的 ID，原先放在 AWS S3 上面的靜態資料夾結構就會 mapping 錯誤，必須要依照新的 ID 去存放相對應的資料夾結構。

由於部分圖片附件在存取時還會同步進行縮圖，如果要用 [s3-cmd](http://s3tools.org/s3cmd-howto) 這類的外部工具直接搬資料，得連各種不同 model 的縮圖定義一起處理，痛苦指數不低。

原先預計要從 HTTP GET request 的 response body 裡直接塞 tempfile ：

```ruby
url = 'https://s3.amazonaws.com'
path = '/BUCKET-NAME/MODEL/ATTACHMENT/000/016/222/original/FILE.jpg'
conn = Faraday.new(:url => url) do |faraday|
  faraday.request  :url_encoded
  faraday.response :logger
  faraday.adapter  Faraday.default_adapter
end

response = conn.get path
model.attachment = response.body
```

一直噴出各式各樣無法存入或者檔案格式錯誤的錯誤訊息。

試過自行重組檔案的檔頭 ：

```ruby
attachment_file = {
  :filename => /^.*\/(.*\..*)$/.match(uri)[1].split('?').first,
  :type => response.headers['content-type'],
  :headers => response.headers,
  :tempfile => response.body
}

model.attachment = attachment_file
```

一樣過不了 Paperclip 的檔案驗證機制。

使用 `File.new(response.body)` 或 `File.new(attachment_file)` 或 `ActionDispatch::Http::UploadedFile.new(attachment_file)` 都解析不了檔案。

撞了好一會牆後，發現 Paperclip 吃 URI object。一切的撞牆只要簡單的：

```ruby
url = 'https://s3.amazonaws.com'
path = '/BUCKET-NAME/MODEL/ATTACHMENT/000/016/222/original/FILE.jpg'

model.attachment = URI.parse("#{url}#{path}")
```

獻給其他可能要做這類奇異操作的破頭工程師們。
