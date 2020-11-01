+++
title = "程設風格 | 早期終止執行"
description = ""
date = 2016-04-15T04:22:50+08:00
tags = ["ruby", "coding"]
draft = false
+++

剛開始寫程式的時候，以為所謂邏輯判斷就是很多個若 A 則 B 包起來的複雜地圖。這樣的程式碼，很容易因為邏輯判斷太過複雜，很難一眼就看到到底目前程式會跑到哪一個 if 分支裡面執行。

```
taget = Dog.new

if target.is_a?(Animal)
  if target.has_four_legs?
    if target.is_a?(Dog)
      puts "wolf!"
    else
      puts "four leg animal can say yeah"
    end
  else
    puts "don't know what that is"
  end
else
  puts "don't know what that is"
end

=> "wolf!"
```

如果透過層層過濾只是要剔除一些情況讓程式不處理，例如有很多個分支其實是「不處理」或是「同樣的簡單處理」，例如上面的 `puts "don't know what that is"` ，可以改用「早期終止」的模式來改寫。

```
def what_it_says(target)
  # 如果不符合條件的參數就會在此提早回傳值
  return "don't know what that is" unless target.is_a?(Animal) && target.has_four_legs?

  # 符合資格的參數才會進入真正的必要判斷
  if target.is_a?(Dog)
    "wolf!"
  else
    "four leg animal can say yeah"
  end
end

taget = Dog.new
puts what_it_says(taget)

=> "wolf!"
```

這個模式可以有效減少邏輯套邏輯的複雜架構，程式碼比較好看出來究竟是要執行什麼事情，也比較好 debug 。

如果要用譬喻法形容，其實就跟人生一樣，別花太多力氣在不重要的事情上面，及早說不。
