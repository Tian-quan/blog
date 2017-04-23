---
title: interview-java基础
date: 2017-04-23 18:55:58
tags: [Java,Interview]
categories: Interview
---

# 多线程
## 多线程并发访问HashMap
* 会造成线程死锁.(原因是并发插入会造成闭散列链表形成闭环,读线程会一直死循环在闭环里)
