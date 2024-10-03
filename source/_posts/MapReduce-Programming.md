---
title: MapReduce Programming
excerpt: 介绍MapReduce的程序设计模式
date: 2024-10-03 16:41:02
tags:
math: true
category_bar: true
categories: Realtime and Big Data Analytics
---

```java
package org.apache.hadoop.io;

import java.io.DataOutput;
import java.io.DataInput;
import java.io.IOException;

public interface Writable {
    void write(DataOutput out) throws IOException;
    void readFields(DataInput in) throws IOException;
}
```