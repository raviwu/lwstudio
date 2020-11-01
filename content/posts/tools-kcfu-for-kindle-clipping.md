+++
title = "工具 | Kindle \"My Clippings.txt\" 的轉檔 gem KCFU"
description = ""
date = 2017-01-28T13:33:09+08:00
tags = ["coding", "ruby", "gem", "kindle", "tools"]
draft = false
+++

Kindle 的剪貼簿功能雖然在 Kindle 上很好用，但真的要抽出來另外處理書摘的時候還真的麻煩，因為 `My Clippings.txt` 就是一個很簡單的純文字檔，每次在 Kindle 上面 hightlight / bookmark / notes 時就會依照書的 title 去新增一段特定格式的文字段落，所以如果同時看很多本的時候，打開 `My Clippings.txt` 會發現裡面就依照 clippings 時序夾雜著來自不同書裡的內容，得再經過一些處理才能分成不同的來源。

原先有找到 [firewood](http://sebpearce.com/blog/firewood/) 這個套件，不過沒辦法在我的電腦上使用 QQ，乾脆就自己刻個輪子吧。

GitHub 上面的 [kindleclippings](https://github.com/georgboe/kindleclippings) 是用 Ruby 寫的，簡單又方便使用，站在小巨人的肩膀上刻了一個 [kcfu](https://github.com/raviwu/kcfu) 來做基本的拆分檔案，另外加配了一個陽春的 Markdown 格式轉換器方便做書摘。

最簡單的用法是開一個資料夾，把你的 `My Clippings.txt` 放到裡面，再寫個小 Ruby 檔：

```ruby
# kcfu.rb
require 'kcfu'

parser = Kcfu::FileUtil.new
parser.parse_file(File.expand_path("#{File.dirname(__FILE__)}/My Clippings.txt"), convert: :markdown)
```

然後安裝 KCFU（沒有安裝 Ruby 的同學若要服用請先安裝 [rbenv](https://github.com/rbenv/rbenv) 跟 ruby ）

```shell
gem install kcfu

# Under folder of your kcfu.rb and 'My Clippings.txt' file
ruby kcfu.rb
```

就搞定了。