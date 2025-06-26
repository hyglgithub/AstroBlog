---
title: Java synchronized 深度指南：锁对象、用法与六种加锁模式详解
published: 2025-06-13
description: Java同步机制的核心是对象监视器锁（monitor lock），通过synchronized实现线程同步。锁对象必须是引用类型，可以是实例对象（this）、类对象（Class）。synchronized有6种写法：3种实例锁（this、实例变量、实例方法）和3种类锁（Class、静态变量、静态方法）。wait/notify机制基于对象Monitor，调用wait()会释放锁并进入等待集合，notify()唤醒线程重新竞争锁。实例锁保护单个实例资源，类锁保护全局类资源。
tags: [java, 锁]
category: '原创'
draft: false 
---

## 一、前置知识

### 1. 同步的概念与锁的作用

* **同步（synchronization）**：多个线程访问共享资源时，必须按照一定顺序进行，以防止数据竞争和不一致问题。
* **锁的作用**：保证同步，让多个线程以串行化方式访问临界区（共享资源/同步代码块）。锁是实现线程同步的关键机制。

### 2. 锁属于对象

* 每个对象都有一个隐式的 **监视器锁（monitor lock）**，这是 Java 内置的同步机制。

### 3. 线程竞争锁，线程是锁的使用者

* 当多个线程访问同一个对象的同步代码块时，只有获取该对象锁的线程才能执行，其他线程会进入等待状态。

### 4. Java 对象的分类

| 类型   | 示例              | 说明                                  |
| ---- | --------------- | ----------------------------------- |
| 实例对象 | `new MyClass()` | 普通对象，常用于实例级别锁（`synchronized(this)`） |
| 类对象  | `MyClass.class` | 每个类在 JVM 中对应一个唯一的 Class，用于类级锁       |

* 类本身（`MyClass.class`）和类的实例（`new MyClass()`）在 JVM 中都是 Object 类型的对象。

---

## 二、synchronized(...) 应该接受什么参数？为什么？

* 参数必须是引用类型对象（Object），原因如下：

  * Java 的同步机制基于对象监视器锁（monitor lock），每个对象都有一个隐式的 Monitor。
  * 锁对象不能是基本类型（如 int、boolean 等）。
  * 线程进入同步代码块前，必须先获取锁对象的 Monitor；如果锁已被其他线程持有，当前线程会阻塞，直到锁被释放。

---

## 三、this 和类名.class 作为锁对象的区别

| 特点    | `synchronized(this)` | `synchronized(MyClass.class)` |
| ----- | -------------------- | ----------------------------- |
| 锁对象   | 实例对象               | 类对象（全局唯一）           |
| 粒度    | 对某个实例加锁              | 对整个类级别资源加锁                    |
| 影响范围  | 同一对象的线程互斥，其他对象不受影响   | 所有对象线程都互斥（无论多少实例）             |
| 多线程访问 | 多个对象可以并发执行           | 所有线程都受影响                      |

---

## 四、synchronized 的 6 种写法

### ① `synchronized(this)` - 当前实例对象作为锁对象

```java
public void instanceLock() {
    synchronized (this) {
        System.out.println("Synchronized on this");
    }
}
```

### ② `synchronized(MyClass.class)` - 当前类对象作为锁对象

```java
public static void classLock() {
    synchronized (MyClass.class) {
        System.out.println("Synchronized on MyClass.class");
    }
}
```

### ③ `synchronized(lock)` - 自定义实例属性作为锁对象

```java
private final Object lock = new Object();

public void customInstanceLock() {
    synchronized (lock) {
        System.out.println("Synchronized on custom instance lock");
    }
}
```

### ④ `synchronized(staticLock)` - 自定义类属性作为锁对象

```java
private static final Object staticLock = new Object();

public static void customStaticLock() {
    synchronized (staticLock) {
        System.out.println("Synchronized on custom static lock");
    }
}
```

### ⑤ `public synchronized void method()` - 当前实例对象作为锁对象

```java
public synchronized void synchronizedMethod() {
    System.out.println("Synchronized method");
}
```

### ⑥ `public static synchronized void staticMethod()` - 当前类对象作为锁对象

```java
public static synchronized void synchronizedStaticMethod() {
    System.out.println("Synchronized static method");
}
```

---

## 五、写法总结

### 实例锁

| 写法                         | 锁对象     | 加锁位置 |
| -------------------------- | ------- | ---- |
| `synchronized(this)`       | 当前实例对象  | 代码块  |
| `synchronized(lock)`       | 自定义实例变量 | 代码块  |
| `public synchronized void` | 当前实例对象  | 实例方法 |

### 类锁

| 写法                                | 锁对象             | 加锁位置 |
| --------------------------------- | --------------- | ---- |
| `synchronized(MyClass.class)`     | 当前类对象             | 代码块  |
| `synchronized(staticLock)`        | 自定义静态变量         | 代码块  |
| `public static synchronized void` | 当前类对象  | 静态方法 |

---

## 六、线程的等待与唤醒机制

### Object 类中的三大方法：

* `wait()`：使当前线程进入等待状态（并释放锁）。
* `notify()`：唤醒一个在此对象监视器上等待的线程。
* `notifyAll()`：唤醒所有在此对象监视器上等待的线程。

### 示例代码：

```java
class WaitNotifyExample {
    private static final Object lock = new Object();

    public static void main(String[] args) {
        // 消费者线程
        Thread consumer = new Thread(() -> {
            synchronized (lock) {
                try {
                    System.out.println("Consumer: Waiting for production to finish.");
                    lock.wait(); // 等待并释放锁
                    System.out.println("Consumer: Production finished. Consuming...");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        // 生产者线程
        Thread producer = new Thread(() -> {
            synchronized (lock) {
                try {
                    System.out.println("Producer: Producing...");
                    Thread.sleep(2000);
                    System.out.println("Producer: Production finished. Notifying consumer.");
                    lock.notify(); // 唤醒一个等待的线程
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        consumer.start();
        producer.start();
    }
}
```

### 代码说明：

1. 锁对象 `lock`：两个线程通过同一个对象进行同步。
2. 消费者线程：

   * 调用 `lock.wait()` 进入等待状态，释放锁。
3. 生产者线程：

   * 模拟生产过程后调用 `lock.notify()` 唤醒消费者线程。

---

## 七、从 Monitor 视角理解 wait() 与 notify() 的本质

### Monitor 简介

在 Java 中，每个对象都关联一个 Monitor（监视器），用于实现线程同步与通信。它包含三个关键部分：

* **Owner**：当前持有锁的线程。
* **Entry List**：等待获取锁的线程队列。
* **Wait Set**：调用 `wait()` 后进入等待状态的线程集合。

---

### wait() 的本质和流程

当线程调用 `wait()` 方法时：

1. **释放锁**：线程放弃对对象锁的持有。
2. **加入 Wait Set**：线程进入等待集合，暂停执行。
3. **等待唤醒**：直到其他线程调用 `notify()` 或 `notifyAll()`。
4. **重新竞争锁**：被唤醒后，进入 Entry List 等待重新获取锁。
5. **恢复执行**：获取锁后，继续执行 `wait()` 之后的代码。

---

### notify() 的本质和流程

当线程调用 `notify()` 或 `notifyAll()`：

1. **唤醒线程**：从 Wait Set 中唤醒一个或全部线程。
2. **进入 Entry List**：被唤醒线程等待重新获取锁。
3. **恢复执行**：获取锁后，继续执行。