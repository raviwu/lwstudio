+++
title = "利用 find_in_batches 在 Rails 做跨資料庫的資料移轉操作"
description = ""
date = 2016-09-02T05:05:44+08:00
tags = ["rails", "db", "coding"]
draft = false
+++

在處理跨資料庫搬動並且需要一筆一筆資料去做各種運算處理時，最直覺的方式，是把表格裡的所有資料通通透過 ORM 撈到 `ActiveRecord::Association` 陣列裡，再用 `each` 去做操作：

```ruby
User.all.each do |user|
    user.email = "assign@new.email"
    ...
end
```

但由於 Rails 會需要先去建立這些 objects，如果資料筆數很多的話，一次通通撈出來會佔用過多不必要的記憶體，可以使用 `find_in_batches` 做批次處理，減少系統 RAM 的負擔。

```ruby
User.find_in_batches do |group|
    group.each do |user|
    user.email = "assign@new.email"
    ...
    end
end
```

但如果要做跨資料庫的處理時，需要特別安插 `ActiveRecord::Base.establish_connection` 讓 Rails 知道當前操作需要連到哪個資料庫，舉例而言，假如想把資料一批一批從 `:origin` 調出來，然後檢查、編輯以後塞到 `:migrated` 資料庫裡，那麼需要在 Query 前去告訴 Rails 目前要連到哪個資料去撈資料：

```ruby
# Tell rails to get data from :origin
ActiveRecord::Base.establish_connection :origin

User.find_in_batches do |group|

    group.each do |user|
    # user variable here are the user object that Rails queried from :origin DB and initialed

    # Tell rails to switch connection to :migrated for data manipulation
    ActiveRecord::Base.establish_connection :migrated

    migrated_user = User.find_by_email(user.email) || User.new(email: user.email)
    migrated_user.assign_attributes(user.attributes.reject{ |key,value| key == 'id' })
    migrated_user.save! # This migrated_user will be saved in :migrated database
    ...
    end

    # Tell rails to switch connection to :origin to query next batch
    ActiveRecord::Base.establish_connection :origin

    # If not assigning the connection before end of the block,
    # find_in_batches will query :migrated database for the next batch with
    # SELECT `users`.* FROM `users`
    #   WHERE (`users`.`id` > 1000)
    #   ORDER BY `users`.`id` ASC
    #   LIMIT 1000

end
```

如此一來，能有效減少 RAM 的負擔，`find_in_batches` 預設一次撈 1000 筆資料，也可以指定批次筆數和 query 的 ID 區間，參考文件 [http://api.rubyonrails.org/classes/ActiveRecord/Batches.html](http://api.rubyonrails.org/classes/ActiveRecord/Batches.html)。
