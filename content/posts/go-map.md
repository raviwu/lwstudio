---
title: "Go | Map"
date: 2020-10-26T14:42:56+08:00
tags: ["coding", "go", "map", "data structure"]
disqus: false
draft: false
---

Go 的 map 是鍵值對的資料結構。

```go
// 宣告一個空白 map
colors := map[string]string{}
// 加入或修改鍵值對
colors["Yello"] = "#cc8500"
// 刪除鍵值（鍵不存在時並不會噴錯誤）
delete(colors, "Red")

// 宣告一個空 map
var numbers map[int]string
numbers[1] = "One"
// panic: runtime error: assignment ot entry in nil map
```

檢查鍵是否存在，如果指定的鍵不存在，會回傳零值給 `value`

```go
value, exists := colors["Blue"]

if exists {
    fmt.Println(value)
} else {
    fmt.Println(value) // ""
}
```

## 迭代 map

```go
numbers := map[int]string{
    1: "one",
    2: "two",
    3: "three",
}

for key, value := range numbers {
    fmt.Printf("Key: %d Value: %s\n", key, value)
}
```

## 使用 map 為函數之參數

傳遞 map 到函數時，並不會另外複製新的 map 物件，函數對 map 參數的操作會反應到其他也指向該 map 的地方。