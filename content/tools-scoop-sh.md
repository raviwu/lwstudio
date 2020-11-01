+++
title = "Windows 裡的類 homebrew 工具： scoop.sh"
date = 2018-08-10T13:39:10+08:00
publishdate = 2018-08-10T13:39:10+08:00
tags = ["coding", "tools", "windows", "package management"]
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

新工作的環境是 Windows 系統，所有的開發工具生態系都要重新摸索。剛開始發現可以用 [Git for Windows](https://gitforwindows.org/) 裝好後的 console 下平常習慣的 bash 指令後，努力想要自己寫一些 shell script 去自動化開發工具跟環境變數的初始化，簡單的情境下好似都還堪用。

可是如果要安裝的套件都要一個一個手刻 script 好像又有些白癡。

因緣際會下被提點去找其他平台上的類 Homebrew 方案，權衡系統權限低落等實際情況下，好像也只剩 [scoop.sh](https://scoop.sh/) 可以用了。

搞了一整天反覆試驗了一陣子，總算是把裝機清單給列了出來，放在[這裡](https://github.com/raviwu/environmentSetups/blob/master/scoop.ps1)。雖然還是要手動輸入指令，但是可以統一管理這些套件，用指令反安裝等已經比之前的原始人狀態進步多了。

被 [oh my zsh](https://ohmyz.sh/) 慣壞眼睛的我，有幸發現可以用 [concfg](https://github.com/lukesampson/scoop/wiki/Theming-Powershell) 稍微把 PowerShell 的顏色弄得順眼一點。所謂山不轉路轉，路不轉人轉，接下來還需要努力爬行。（握拳）
