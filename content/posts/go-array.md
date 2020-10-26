---
title: "Go | 陣列（Array）"
date: 2020-10-26T11:36:46+08:00
tags: ["go", "array", "coding"]
disqus: false
draft: false
---

Go 的陣列是連續空間內儲存了固定數目之相同型別物件的容器。當使用 `var array [5]int` 宣告時，陣列內儲存的型別或者長度都不能再改變，如果需要更多的空間來儲存新的物件，需要宣告一個新的陣列後把原先的值搬過去新的陣列。

在 Go 語言中宣告新的變數且無賦值時，這些變數所指向的物件都會以其零值（Zero Value）初始化。有關於 Go 的零值，請參考[這邊](https://dave.cheney.net/2013/01/19/what-is-the-zero-value-and-why-is-it-useful)。常用的零值：

|類型|零值|
|---|---|
|數值|0|
|布林值|false|
|字串|""|

## 初始化陣列的同時賦值

```go
// 宣告一個 array1 變數的 int 陣列，長度為 5 且值依序分別為 1,2,3,4,5
array1 := [5]int{1,2,3,4,5}

// 宣告一個 array2 變數的 int 陣列，長度為 5 且值依序分別為 0,1,2,0,0
array2 := [5]int{1: 1, 2: 2}

// 宣告一個 array3 變數的 int 陣列，長度為取決於賦值內容
// 如以下 array3 的長度為 3
array3 := [...]int{1,2,3}
```

## 陣列的操作

```go
// 宣告一個 array 變數的 int 陣列，長度為 3 且值依序分別為 1,2,3
array := [3]int{1,2,3}

// 把 index 2 的值改為 5
array[2] = 5
```

## 使用指標為陣列的值

初始化 `array := [3]*int{0: new(int), 1: new(int)}` 時：

|index|0|1|2|
|-----|---|---|---|
| array[index] | 0xc0000b6020 | 0xc0000b6028 | nil |
| *array[index] |0| 0| `error: cannot use assignment (array[i]) = (nil) as value` |

試試看改變陣列指標的值 `*array[0] = 100` ：

|index|0|1|2|
|-----|---|---|---|
| array[index] | 0xc0000b6020 | 0xc0000b6028 | nil |
| *array[index] |0| 100 | `error: cannot use assignment (array[i]) = (nil) as value` |

這邊可以注意到，指標的記憶體位址是一樣的，但值改變了。

假如想要替現在為 `nil` 的位置指定值，首先要在記憶體裡做出一個指標。

```go
i := 200
array[2] = &i
```

透過 `i := 200` 的方式宣告了 `i` 以後，使用 `&i` 取得 `i` 的指標並賦值給 `array[2]` 後，會變成：

|index|0|1|2|
|-----|---|---|---|
| array[index] | 0xc0000b6020 | 0xc0000b6028 | 0xc0000b6038 |
| *array[index] |0| 100 | 200 |

假如另外再賦值另一個變數 `j` 時，指標位址會變：

```go
j := 200
array[2] = &j
```

這邊例外宣告了變數 `j` 給整數 200，其實是另外在記憶體裡找了一塊地方塞值 200 給指標 `j`，到此， `i` 跟 `j` 的位置不同，但值相同。

```go
i := 200
j := 200
k := j

i == j // true
k == j // true

&i == &j // false
&k == &j // false

k = 300
k == j // false
```

突然跳來實驗一下 Go 的指標，可以發現每 `:=` 了一個指標，都是在記憶體裡拿一塊新的區域來存值，這幾個指標都是不同位置，所以假若 `*k` 被改值時不會影響到 `i` 或 `j` 這兩個指標所指向的值。

|index|0|1|2|
|-----|---|---|---|
| array[index] | 0xc0000b6020 | 0xc0000b6028 | 0xc0000b6058 |
| *array[index] |0| 100 | 200 |

重新指定一個指標給 `array[2] = new(int)` 也是一樣的意思：

|index|0|1|2|
|-----|---|---|---|
| array[index] | 0xc0000b6020 | 0xc0000b6028 | 0xc0000b6078 |
| *array[index] |0| 100 | 0 |

要注意的是，如果複製以指標為值的陣列時，指標的位置會一樣：

```go
array1 := [2]*int{0: new(int), 1: new(int)}
array1[0] = 1
array1[1] = 2

array2 := array1
```

可以發現 `array1` 跟 `array2` 裡頭放的指標，地址都是一樣的。

|index|0|1|
|-----|---|---|
| array1[index] | 0xc0000b6020 | 0xc0000b6028 |
| array2[index] | 0xc0000b6020 | 0xc0000b6028 |
| *array1[index] |1|2|
| *array2[index] |1|2|

當然，這就表示，如果改動指標指向的值，兩個陣列都會被影響，請注意！

## 傳送陣列給函數

由於 Golang 的函數是傳值（passed by value），所以把陣列直接傳進去函數裡面，對於程式效能會有負面影響。

可以試試看下面的實驗：

```go
// 宣告一個 8MB 的陣列
var array [1e6]int

// 把陣列直接傳到 foo 函數裡
foo(array)

// foo 直接接收陣列
func foo(array [1e6]int) {
    ...
}
```

每一次 `foo` 這個函數被呼叫，就會有 8MB 的空間在 stack 被取用。然後傳入的陣列會被複製到這個新取用的空間裡。即使你的機器可以處理這樣的負載，但其實有更好的方式。

### 使用指標（Pointer）作為參數

```go
var array [1e6]int

foo(&array)

func foo(array *[1e6]int) {
    ...
}
```

這麼做，呼叫 `foo` 的時候就不會浪費額外的記憶體空間去複製傳入的值，而是使用 `array` 這個已經建立好的陣列。

但這麼做要小心函數是否對於這個參數的值做改變，由於傳入的是指標，所以對於陣列的值做任何改變時，同一個記憶體中的數值會被更改。
