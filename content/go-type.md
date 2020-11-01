+++
title = "Go | 型別（Type）"
date = 2020-10-28T12:05:03+08:00
publishdate = 2020-10-28T12:05:03+08:00
tags = ["coding", "go", "type", "struct", "function"]
draft = false

description = ""
summary = ""
keywords = []

[amp]
    elements = []

[author]
    name = "Ravi Wu"
    homepage = "https://raviwu.github.io"

[image]
    src = ""

[sitemap]
    changefreq = "monthly"
    priority = 0.5
    filename = "sitemap.xml"
+++

Go 的型別（Type）可以讓編譯器知道兩樣資訊：

1. 需要使用的記憶體大小
2. 這些記憶體所代表的內容

以內建的型別為例：

|型別|記憶體大小|內容|
|----|----|----|
|int64|8 bytes|整數|
|float32|4 bytes|[IEEE-754](https://en.wikipedia.org/wiki/IEEE_754) 浮點數|
|bool|1 byte|true OR false|

有一些型別所代表的內容會跟著 build 機器的不同架構有所差異，例如同樣的 `int` 在 64 位元電腦裡佔 8 bytes 但在 32 位元電腦裡只佔 4 bytes。

## 自定義型別

Go 容許自定義型別，最簡單的宣告方式為 struct

```go
type user struct {
    name string
    email string
    ext int
    isAdmin bool
}
```

使用 `var ravi user` 可以宣告一個 `ravi` 的變數，代表一個 `user` 型別。宣告的當下，型別的欄位值會使用各欄位之型別的零值。

當需要直接復值時也可以使用以下語法：

```go
tom := user {
    name: "Tom",
    email: "tom@foo.bar",
    ext: 321,
    isAdmin: true,
}
```

或者也可以使用 `tom := user{"Tom", "tom@foo.bar", 321, true}` 來宣告，通常都會把所有值放在同一行以縮減最後一個值的逗點。用單行宣告時，變數順序至關重要，需要依照自定義欄位的順序提供。

已知自定義型別也能被當成欄位使用：

```go
type admin struct {
    person user
    level string
}
```

## 自定義型別的函數宣告

型別函數宣告使用 `(u user)` 或 `(u *user)` 的時機：

```go
// Value Receiver
func (u user) notify() {
    send(u.email)
}

// Pointer Receiver
func (u *user) changeEmail(email string) {
    u.email = email
}
```

如果函數取值做操作，不需要改變原物件欄位時，使用傳值的方式 `(u user)` 宣告；如果函數需要針對物件作操作，更改欄位值的內容時，使用傳指標的方式 `(u *user)` 去做宣告。

### Go 的貼心動作

```go
ravi := &user("Ravi", "ravi@foo.bar")
ravi.notify()
```

即使上面的 `ravi` 變數透過 `&user` 被指定為指標，對指摽 `ravi` 呼叫宣告傳值的 `notify()` 仍可以編譯執行。原因在於 Go 編譯器在函數呼叫時另行處理為 `(*ravi).notify()` 並且複製了一份 `*ravi` 物件進函數處理。

同樣地：

```go
tomas := tomas("Tomas", "tomas@foo.bar")
tomas.changeEmail("newTomas@foo.bar")
```

也可以編譯成功，因為 Go 在做函數呼叫時實際上是長這樣： `(&tomas).changeEmail("newTomas@foo.bar")`

## Go 的內建原始型別（Primative Types）

針對數字、字串和布林值等 Go 的內建型別，當對其值進行操作時，一律是對新的副本做操作。

## Go 的參照型別（Reference Types）

像是[切片](/posts/go-slice)、[map](/posts/go-map)、channel、介面（interface）與函數（function）等參照型別，在宣告變數給這些型別的當下，只有被稱為 header 的值產生。嚴格來說，字串也是屬於參照型別。這些不同型別的 header 值，儲存了指向底層資料結構記憶體位址的指標（Pointer）。每個不同的參照型別除了這個指標之外還另外儲存了管理底層資料結構所需的訊息。（例如切片的 Length 與 Capacity）

因為這個特性，在使用參照型別時，一律都是針對新的副本做操作。要注意的是，由於底層的資料結構可能與其他指標共用，針對資料變動需要特別小心。

## Struct 型別

Struct 可以包含原始或非原始型別，在定義過程應該根據 Struct 欄位或者是其設計目的來決定這個 Struct 應該適用傳值或是傳指標式操作。

以標準庫的範例來看：

### Time

```go
type Time struct {
    sec int64
    nsec int32
    loc *Location
}

func Now() Time {
    sec, nsec := now()
    return Time{sed + unixToInternal, nsec, Local}
}
```

Time 的屬性多是原始性質，所以大多數的內建函數是回傳一個新的 Time。

### File

```go
type File struct {
    *file
}

type file struct {
    fd int
    name string
    dirinfo *dirInfo
    nepipe int32
}
```

從定義看，File 型別根據定義，屬於參照型別。

```go
func Open(name string) (file *File, err error) {
    return OpenFile(name, O_RDONLY, 0)
}
```

Go 的標準庫實作回傳一個 File 的指標。

## 介面（Interfaces）

多型（Polyporphism）是型別（Type）透過實作不同介面（Interface）來產生不同的行為。

介面定義了行為，而實際上的實作內容要在自定義型別上界定。

把值指定給 `notifer` 介面的變數時：

```
             var n notifier
  notifier   n = user{"Ravi"}
  介面的值                     iTable
 [位址]
 iTable  -------------------> [Type(user)]
 ------                       ------------
 [位址]       儲存值             Method set
  user   --> [User]
```

把指標指定給 `notifer` 介面的變數時：

```
             var n notifier
  notifier   n = &user{"Ravi"}
  介面的值                     iTable
 [位址]
 iTable  -------------------> [Type(*user)]
 ------                       ------------
 [位址]       儲存值             Method set
  user   --> [User]
```

### 函數集（Method sets）

函數集定義了介面實作是否合規。假如實作時只宣告指標作為 Method call 的 Receiver 時，那只有指標實作了介面，呼叫介面方法時需傳入指標。這個限制的理由是因為單就值本身並不能總是取得位址。

```
Values          Methods Receivers
---------------------------------
  T                (t T)
  *T               (t T) and (t *T)

Methods Receivers      Values
---------------------------------
 (t T)                  T and *T
 (t *T)                 *T
```

當我們傳值給函數時，Method Receiver 只有辦法拿到值本身，但如果是傳指標，Method Receiver 可以除了取得指標以外，還能透過指標拿到值。

反過來看，當 Method Receiver 定義為值的時候，傳入的參數可以是指標或者是值；但如果 Method Receiver 定義為取得指標的話，傳入的參數只能是指標。

再回顧一次，因為，值本身並不能總是解析出確切的位址。

## 嵌入型別（Embedded Type）

Go 允許使用者復用現有型別，而且使用者可以在復用時覆寫型別的行為。以下方程式碼為例：

```go
type user struct {
    name string
    email string
}

func (u *user) notify() {
    sendEmail(u.email)
}

type admin struct {
    user // 內嵌型別
    level string
}
```

`user` 是 `admin` 的內嵌型別，所以如果初始化 `ad := admin { ... }` 後，可以透過 `ad.user.notify()` 來呼叫綁定給 `user` 類別的 `notify()` 函數。特別的是，也可以直接使用 `ad.notify()` 的方式呼叫函數。

來試試看用介面的方式來呼叫函數：

```go
type notifier interface {
    notify()
}

type user struct {
    name string
    email string
}

func (u *user) notify() {
    sendEmail(u.email)
}

type admin struct {
    user // 內嵌型別
    level string
}

func sendNotification(n notifer) {
    n.notify()
}

func main() {
    ad := admin{
        user: user{
            name: "Ravi",
            email: "ravi@foo.bar",
        },
        level: "super",
    }

    sendNotification(&ad)
}
```

從這個範例可以看出，嵌入型別的同時，外型別同時也具有內型別實作的介面。如果想要覆寫內型別的介面行為呢？

```go
func (a *admin) notify() {
    fmt.Printf("Sending admin email: %v\n", a.email)
}
```

由於外型別自己也實作了 `notify()`，此時 `ad.user.notify()` 和 `ad.notify()` 所引用的函數就會不同了：只有特地援用內型別的寫法才會引用到原先內型別的實作。

如果是丟進介面函數，同樣的邏輯下，`sendNotification(&(ad.user))` 和 `sendNotification(&ad)` 也會呼叫到不同的實作。

## 套件的匯出（Exporting）與非匯出（Unexporting）定義

套件內部元件的可見性用來控制函數或型別能否被外部引用。Go 使用命名規則來判定是否可被外部引用：

1. 使用小寫開頭的定義為內部定義，無法被外部引用
2. 使用大寫開頭的定義為公開介面，可以被外部直接引用

此原則套用在：

1. Interface
2. Type
3. Field of Type
4. Function
5. Var / Const

要注意的是，型別的欄位可以選擇性匯出：

```go
// package auth
type User struct {
    Name string
    email string
}

// package test

import "auth"

func test() {
    u := auth.User{
        Name: "Ravi",
        email: "ravi@foo.bar",
        // error comes here: unknown auth.User field 'email'...
    }
}
```
