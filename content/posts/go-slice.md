+++
title = "Go | 切片（Slice）"
description = ""
date = 2020-10-26T13:14:21+08:00
tags = ["go", "coding", "slice", "data structure"]
draft = false
+++

切片（Slice）是操作其指向之[陣列（Array）](/posts/go-array/)的物件。

```
[指標|長度|最大長度] // 切片
  |
[1,2,3,4,5] // 陣列
```

## 切片的初始化

```go
// 宣告一個長度與最大長度都是 5 的字串陣列
slice1 := make([]string, 5)

// 宣告一個長度為 3 最大長度為 5 的整數陣列
slice2 := make([]int, 3, 5)
```

長度不能大於最大長度，不然編譯器會噴錯誤。一般來說，宣告切片的同時也會同時賦值：

```go
// 宣告一個長度與最大長度為 3 的切片
slice1 := []string{"One", "Two", "Three"}

// 宣告一個長度與最大長度為 100 的切片
slice2 := []string{99: ""}
```

## 空切片（nil slice）與空白切片（empty slice）

不管是空切片還是空白切片，標準函式庫裡的 `append` 、 `len` 跟 `cap` 的使用行為都一樣。

### 空切片（nil slice）

在 Go 程式中，空切片很常出現，通常被當成不存在資料時的回傳值，例如 `return nilSlice, error`。

空切片的宣吿方式：

```go
var slice []int
```

空切片長成這樣：

|位置|0|1|2|
|--------|-|-|-|
|用途|指標|長度|最大長度|
|值|`nil`|0|0|

### 空白切片（empty slice）

空白切片通常是用來表達零資料集合，例如查詢資料庫後回傳零筆資料。空白切片的指標指向空陣列，所以並不會另外佔用記憶體。

```go
slice := make([]int, 0)
slice := []int{}
```

## 操作切片

```go
slice := []int{100,200,300,400,500}
newSlice := slice[1:3]
```

看一下記憶體內的樣子：

```
   slice := []int{100,200,300,400,500}
 [ 指標 | 5 | 5 ]
    |
   [0]     [1]     [2]     [3]     [4]
[  100  |  200  |  300  |  400  |  500  ] // 底層陣列
            |
         [ 指標 | 2 | 4 ]
           newSlice := slice[1:3]
```

雖然 `slice` 跟 `newSlice` 都指向同一個陣列，但這兩個切片的世界觀是不同的。對於一個最大長度為 `k` 的切片 `slice[i:j]`：

```
Length:   j - i
Capacity: k - i
```

對 `newSlice` 而言，陣列裡的第一個元素甚至是不存在的。

需要注意的是，由於切片是指向某個底層陣列，所以如果有其他的切片對於這個陣列有做任何操作，操作結果將會同時反映到其他也指向此陣列的切片。

## 長度與最大長度

即使最大長度允許，但切片不能存取大於長度的位址。

```go
slice := []int{1,2,3,4,5} // [ pointer | 5 | 5 ]

newSlice := slice[1:3] // [ pointer | 2 | 4]

newSlice[2] = 45

// runtime error: index out of range [2] with length 2
```

### append 切片後的長度在原陣列長度 -> 新切片指向原陣列

```go
slice := []int{100,200,300,400,500}

newSlice := slice[1:3]

newSlice := append(newSlice, 600)
```

針對擴長後之新切片的操作會反映到原陣列

```
   slice := []int{100,200,300,400,500}
 [ 指標 | 5 | 5 ]
    |
   [0]     [1]     [2]     [3]     [4]
[  100  |  200  |  300  |  600  |  500  ] // 底層陣列
           | |
           | |
        ---- ----------------------------
        |                               |
     [ 指標 | 2 | 4 ]                 [ 指標 | 3 | 4 ]
       newSlice := slice[1:3]          newSlice := append(newSlice, 600)
```

### append 切片後的長度超出原陣列長度 -> 新切片會指向複製原陣列之值的新陣列

`append` 函數會調節最大長度的擴長，針對長度 1000 以下的切片，會以兩倍增長，對於長度 1000 以上的切片則使用 1.25 的倍數做最大長度擴增。

```go
slice := []int{100,200,300,400,500}

newSlice := append(newSlice, 600)
```

來看看記憶體的狀態：

```
   slice := []int{100,200,300,400,500}
 [ 指標 | 5 | 5 ]
    |
   [0]     [1]     [2]     [3]     [4]
[  100  |  200  |  300  |  400  |  500  ] // 底層陣列
            |
         [ 指標 | 2 | 4 ]
           newSlice := slice[1:3]

   newSlice := append(newSlice, 600)
 [ 指標 | 6 | 6 ]
    |
   [0]     [1]     [2]     [3]     [4]     [5]
[  100  |  200  |  300  |  400  |  500  |  600  |  0  |  0  |  0  |  0  ]
// 新的底層陣列
```

## 三參數指定切片

```go
source := []int{100,200,300,400,500}
slice := source[2:3:4]

slice[0] // starting from source[2]
// Legth:    3 - 2 = 1
// Capacity: 4 - 2 = 2

erroredSlice := source[2:3:6]
// Runtime Error: panic: runtime error: slice bounds out of range
```

使用同樣的長度與最大長度的好處在於，當進一步透過 append 去拓展切片時，由於原 slice 的最大長度已用盡， append 會在底層新增一個陣列並且把值複製過去新的陣列，後續新增的值只會加到這個新陣列，不會對原陣列有變動。

## range 迭代切片時，區塊內的值為新值

``` go
slice := []int{10, 20, 30, 40}

for index, value := range slice {
    fmt.Printf("Value: %d Value-Addr: %X ElemAddr: %X\n",
        value, &value, &slice[index])
}

// Output:
// Value: 10 Value-Addr: 10500168 ElemAddr: 1052E100
// Value: 20 Value-Addr: 10500168 ElemAddr: 1052E104
// Value: 30 Value-Addr: 10500168 ElemAddr: 1052E108
// Value: 40 Value-Addr: 10500168 ElemAddr: 1052E10C
```

`range` 總是會從 0 開始迭代，如果有特殊的起始需求，可以使用一般的 `for` 操作：

```go
slice := []int{1,2,3,4}

for index := 2; index < len(slice); index++ {
    fmt.Printf("Index: %d Value: %d\n", index, slice[index])
}
```

## 傳送切片為函數參數

由於切片所佔用的記憶體很少，和陣列不同，直接傳送一個大尺寸的切片進去函數裡並不會增加太多效能負擔。
