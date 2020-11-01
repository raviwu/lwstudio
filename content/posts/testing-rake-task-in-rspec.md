+++
title = "在 RSpec 裡測試 Rake Task"
description = ""
date = 2017-05-07T13:36:08+08:00
tags = ["coding", "ruby", "testing", "rspec", "rake"]
draft = false
+++

最近被 Rake Task 的測試設定搞得一頭霧水，簡單記錄一下測試 Rake Task 的測試設定，以及各種鬼打牆的血淚史：

## TL;DR

為了避免各種 task 載入、執行狀態等相互干擾導致 test case 會偶發性失敗，每一次執行 test case 時就去做「載入」、「卸載」會比較沒有鬼打牆狀況出現。

```ruby
# spec_helper.rb

config.before(:each, rake: true) do
Rails.application.load_tasks
end

config.after(:each, rake: true) do
Rake::Task.clear
end
```

把 `Rails.application.load_tasks` 放在 `before(:each)` 確保每次載入 test case 前都有正確 load 到要測試的那隻 task ，並在 `after(:each)` 時用 `Rake::Task.clear` 去清空剛剛載入的 Rake Task。

透過 `rake: true` 這個 `flag` 可以避免其他不相關的單元測試也去載入 Rake Task。

透過每個 each 都做載入跟卸載 Rake Task 後，就可以在 test case 單純使用 `Rake::Task['task_name'].invoke` 來手動執行 Task 而不用另外去作載入或卸載。

```ruby
# spec/lib/tasks/example_task_spec.rb
require 'rake'

describe 'example_task test', rake: true do
    # ...
    it 'does something' do
    Rake::Task['example_task'].invoke
    expect(result).to eq(something)
    end
end
```

## 血淚史

之所以最後還是決定要每次 test case 就去載入，實在是因為太常出現鬼打牆的狀況了：

### 單一檔案裡 Rake::Task 只能被 invoke 一次

有多個需要 invoke 的 task 時就會有些 example 沒能成功執行 task

之前寫 Rake Task 測試時，都單獨在 `*_spec.rb` 裡面直接載：

```ruby
require 'rake'
# ...
describe 'some rake test' do
    Rails.application.load_tasks

    it 'does something' do
    Rake::Task['task_name'].invoke
    end
end
```

這個寫法每個測試檔案單獨跑的時候都沒有太大問題，因為每次載入檔案時就會透過 `Rails.application.load_tasks` 去載入所有定義在 `lib/tasks/*.rb` 裡面的 Rake Task，並且透過 `Rake::Task['task_name'].invoke` 來手動 invoke 測試目標。

但是！如果要在單一的測試裡面有多個 example 需要 invoke Rake Task 的話，單獨用 `invoke` 就會時不時出現某些 example 沒有正確執行 Rake Task 的狀況。

之前蠢蠢地只用

```ruby
Rake::Task['task_name'].reenable
Rake::Task['task_name'].invoke
```

或是

```ruby
Rake::Task['task_name'].execute
```

去強制執行那隻 Rake Task 來通過測試。然後就繼續過著美好的日子直到⋯⋯

### Rake Task 被 invoke 了超出預期的次數

強制 invoke task 的話，在單一檔案跑測試都沒問題，但如果是多個 rake task spec 一起跑的時候，就會一直出現鬼打牆的狀態，明明只呼叫 `Rake::Task['task_name'].execute` 一次卻會被 invoke 多一次。（眼神死

在推測多個 test file 相互干擾為主因的情況下，翻找老半天，後來才找到是因為每一隻 `*_spec.rb` 都豪氣地 `Rails.application.load_tasks` 後就載後不理，導致某些 task 被 define 了多次，所以跑 `Rake::Task['task_name'].execute` 的時候就會多跑那些被重複定義的 task 。

在「每個 test 檔案去載入與卸載」和「把載入卸載寫在 `spec_helper.rb` 」這兩個方案之間，我後來是選擇了後者，看起來乾淨一滴滴。（汗
