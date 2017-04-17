---
title: Java并发编程--同步
date: 2017-04-12 08:38:47
tags:
---
# 线程基础

## Java线程状态图
![可重入锁](/img/Java并发编程/Java线程状态图.png)
[图片出处](https://my.oschina.net/mingdongcheng/blog/139263)

## sleep
睡眠,放弃CPU使用,让其他线程去竞争CPU时间片.但是会持有锁.

## yield
让步,放弃CPU使用,自己和其他进程一起参与竞争CPU时间片,会持有锁.

## wait
释放锁并等待唤醒.如果没有设置等待时间或者,没有线程调用notify或notifyAll,那么该线程会一直在等待队列中.

## notify
唤醒,唤醒某个等待该锁的线程.即使调用notify方法,也会等该线程退出临界区才释放锁并唤醒某个等待该锁的线程

## notifyAll
唤醒所有,唤醒所有等待该锁的线程,但是只要一个锁能竞争到锁,进入临界区.

## join
加入,假如在main线程中，调用thread.join方法，则main方法会等待thread线程执行完毕或者等待一定的时间。
如果调用的是无参join方法，则等待thread执行完毕，如果调用的是指定了时间参数的join方法，则等待指定的时间.

## interrupt
中断,调用interrupt方法可以使得处于阻塞状态的线程抛出一个InterruptedException，
也就说，它可以用来中断一个正处于阻塞状态的线程;非阻塞的线程是无法被中断的.
另外，通过interrupt方法和isInterrupted()方法来停止正在运行的线程.

## volatile
保证线程的可见性.使用volatile必须具备以下2个条件：
    1)对变量的写操作不依赖于当前值.
    2)该变量没有包含在具有其他变量的不变式中


# 可重入锁
![可重入锁](/img/Java并发编程/可重入锁.png)

# 同步器
![同步器](/img/Java并发编程/同步器.png)