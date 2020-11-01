+++
title = "使用複合條件來判斷是否進入下一輪 Java 迴圈"
description = ""
date = 2020-09-10T13:41:31+08:00
tags = ["coding", "java", "string", "early break"]
draft = false
+++

在 Java 迴圈寫法中，假如我想要透過一個外部判斷來提前中斷這個迴圈，之前我習慣寫：

```java
public class Test {
    public static void main(String[] args) {
        int [] ints = {1,2,3,4,5};
        boolean continueCond = true;
        for (int i=0; i<ints.length; i++) {
            continueCond = ints[i] < 4;
            if (!continueCond) break;
            System.out.println(ints[i]);
        }
    }
}
```

但這樣寫其實有點冗，今天看到在 [Functional Interfaces in Java](https://www.apress.com/gp/book/9781484242773) 裡的一個範例寫法：

```java
public class Test {
    public static void main(String[] args) {
        int [] ints = {1,2,3,4,5};
        boolean continueCond = true;
        for (int i=0; i<ints.length && continueCond; i++) {
            continueCond &= ints[i] < 4;
            System.out.println(ints[i]);
        }
    }
}
```

但這樣寫的行為跟使用 `if (!continueCond) break;` 強制跳出迴圈有些差異，由於 `for` 裡面是要走完全部才會跳出，所以第二種寫法依然會把 4 給印出來。以這個範例來說，變成要把需要跳開的動作包在一個條件裡：

```java
public class Test {
    public static void main(String[] args) {
        int [] ints = {1,2,3,4,5};
        boolean continueCond = true;
        for (int i=0; i<ints.length && continueCond; i++) {
            continueCond &= ints[i] < 4;
            if (continueCond) System.out.println(ints[i]);
        }
    }
}
```

本來想說好像是會比較簡潔一點，但發現改成後面這種寫法以這個範例來說並沒有特別的好處，以書上的情境可能會更適用：

```java
static boolean isLowerCase(String s) {
    boolean result = true;
    for (int i=0; i<s.length() && result; i++) {
        result &= Character.isUpperCase(s.charAt(i));
    }
    return result;
}
```

如果迴圈本身只需要找到第一個符合某個條件的情況就可提前中斷後續的判讀，並且迴圈本身沒有 side effect 的話，這樣做可以免除寫那一行 `break;` 的篇幅。參考參考。