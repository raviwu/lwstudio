+++
title = "泛型 Java Functional Interface 的特規化"
description = ""
date = 2020-09-11T13:44:04+08:00
tags = ["coding", "java", "specilization", "functional interface"]
draft = false
+++

如果某個特殊型別的 Functional Interface 常常被使用時，有時候直接宣告型別會比較方便引用。可以透過兩種方式來做特規，介面與抽象類別：

Generic Functional Interface

```java
@FunctionalInterface
public interface TwoArgsProcessor<X> {
    X process(X arg1, X arg2);
}
```

介面（Interface）

```java
@FunctionalInterface
public interface TwoIntsProcessor
    extends TwoArgsProcessor<Integer> {
}
```

抽象類別（Abstract Class）

```java
abstract class TwoIntsProcessorAbstract
    implements TwoArgsProcessor<Integer> {
}
```