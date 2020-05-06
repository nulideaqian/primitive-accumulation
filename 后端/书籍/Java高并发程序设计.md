#  Java高并发程序设计

## 第1章 走入并行的世界

### 1.1 何去何从的并行计算

1. Linus说：“在图像处理和服务端程序中是可以使用并发编程的。”
2. 如何让多个CPU有效并且正确的工作成为了一门技术。

### 1.2 你必须知道的几个概念

1. **同步和异步**

| 名称 | 值                                                           |
| ---- | ------------------------------------------------------------ |
| 同步 | 方法调用开始后，必须等待方法结束，才能继续后续行为。         |
| 异步 | 方法调用后，立刻返回，调用者继续后续的行为。而异步的方法一般在另一个线程中运行。 |

2. **并发和并行**

| 名称 | 值                                                         |
| ---- | ---------------------------------------------------------- |
| 并发 | 并发偏重多个任务交替执行，而多个任务之间有可能还是串行的。 |
| 并行 | 并行时多个任务是真的同时执行                               |

3. 真实系统中，如果只有一个CPU，多进程或多线程都是并发的，毕竟一个CPU同时只能执行一个指令，而是由操作系统进行任务的切换。

4. **临界区**

   - 概念：表示一种公共资源或者共享数据，可以被多个线程使用。
   - 但是每一次只有一个线程使用它，一旦临界区资源被占用，其他线程要想使用这个资源必须要等待。

5. 阻塞和非阻塞

   > 它们通常用来形容多线程间的相互影响。

   **阻塞**：比如一个线程占用了临界区的资源，那么其他所有需要这个资源的线程都必须在这个临界区中等待。等待会导致线程挂起，这个情况就是阻塞。

   **非阻塞**：没有一个线程可以妨碍其他线程工作，所有的线程都会尝试不断向前执行。

6. 死锁、饥饿和活锁

   **死锁**：所有的线程都在等待其他线程释放资源，导致程序无法执行下去。

   **饥饿**：指某一个或者多个线程因为种种原因无法获得所需要的资源，导致一直无法执行。比如：他的线程优先级太低，而高优先级的线程又不停的占用所需的资源，导致其一直无法工作。

   **活锁**：多个线程主动释放自己的资源，从而也导致没有线程可以获取所有的资源而工作。

### 1.3 并发的级别

1. **阻塞**

   一个线程是阻塞的，那么在其他线程释放资源之前，当前线程则无法继续执行。比如我们使用**synchronized**关键字或者**重入锁**时，得到的就是阻塞的线程。

2. **无饥饿（Starvation-Free）**

   如果线程间是有优先级的，那么线程调度的时候总是会倾向于优先满足高优先级的线程。也就是说如果在有高优先级的线程不停插队的情况下，部分低优先级的线程任务始终无法执行。

   如果锁是公平的，按照排队的原则，所有的线程都有机会执行。

3. **无障碍（Obstruction-Free）**

   可以理解为乐观锁。

   它是最弱的一种非阻塞调度。多个线程都可以无障碍的执行，都可以直接进入临界区。不过如果检测到冲突，则立即对自己所做的修改进行回滚，确保数据安全。

   它认为多个线程之间很有可能不发生冲突，认为概率不大。因此大家都应该无障碍的执行，但是一旦检测到冲突，就应该进行回滚。

   可增加“一致性标记”

4. **无锁（Lock-Free）**

   无锁的并行都是无障碍的。在无锁的情况下，所有的线程都能尝试对临界区进行访问，但不同的是，无锁的并发保证必然有一个线程能够在有限的步骤内完成操作并离开临界区。

5. **无等待（Wait-Free）**

   他要求所有的线程都必须在有限步骤内完成。

   一个典型的实现RCU（Read Copy Update）：它的基本思想，对数据的读可以不加控制。因为所有的读线程都是无等待的。但是写数据的时候，先取回原始数据的副本，接着修改后，在合适的时机回写数据。

### 1.4 有关并行的两个重要定律

1. 为什么要并行？第一，为了获得更好的性能；第二，由于业务模型的需要，确实需要多个执行实体。
2. Amdahl定律：
   - **加速比** = 优化前系统耗时 / 优化后系统耗时
   - CPU数量越多，串行化比例越低，则优化效果越好
3. Gustafson定律：如果串行化比率很小，并行化比例很大，那么只要不停的累加处理器，就能获取更快的速度。

### 1.5 回到Java：JMM

> 并发程序比串行程序复杂的一个重要原因：并发程序中数据访问的一致性和安全性将会受到严重的挑战。
>
> JMM的关键技术都是围绕多线程的原子性、可见性和有序性来建立的。

1. 原子性：

   **概念**：一个操作时不可中断的。即使时在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。

   ```java
   // 如下代码示例，在本机器上执行，结果不是200000 而是199996，说明i++的操作不是原子的
   package com.goahead.chapter01;
   
   public class TestAtomicity {
   
       private static int globalInt = 0;
   
       public static void main(String[] args) {
           Runnable task1 = () -> {
               for (int i = 0; i < 100000; i++) {
                   globalInt++;
                   System.out.println(Thread.currentThread().getName() + " task1 ---- " + globalInt);
               }
           };
           Thread thread1 = new Thread(task1);
   
           Runnable task2 = () -> {
               for (int i = 0; i < 100000; i++) {
                   globalInt++;
                   System.out.println(Thread.currentThread().getName() + " task2 ---- " + globalInt);
               }
           };
           Thread thread2 = new Thread(task2);
   
           thread1.start();
           thread2.start();
   
       }
   
   }
   ```

2. 可见性（Visibility）

   **概念**：指当一个线程修改了某一个共享变量的时候，其他的线程是否能够立即知道这个修改。

3. 有序性（Ordering）

   为了降低CPU流水线的停顿次数，会进行指令重排序。

   对于一个线程来说，它看到的指令执行顺序一定是一致的。指令的重排序的基本前提，需要保证串行语义的一致性。指令重排不会使串行的语义逻辑发生问题。

   **注意**：指令重排保证了串行语义的一致，但是没有义务保证多线程间的语义也一致。

4. 哪些指令不能重排：Happen-Before规则

   - 程序顺序原则：一个线程内保证语义的串行性。
   - volatile规则：volatile变量的写先于读，这保证了volatile变量的可见性。
   - 锁规则：解锁必然发生在随后的加锁前
   - 传递性：A先于B，B先于C，那么A一定先于C
   - 线程的start方法先于它的每一个动作
   - 线程的所有操作先于线程的终结（Thread.join()）
   - 线程的中断（interrupt()）先于被中断的代码
   - 对象的构造函数的执行、结束先于finalize()方法

## 第2章 Java并行程序基础

### 2.1 有关线程你必须知道的事

1. 进程是系统进行资源分配和调度的基本单位，是操作系统结构的基础。
2. 进程是程序的实体。
3. 线程是轻量级的进程，是程序执行的最小单位。使用多线程而不是多进程去进行并发程序设计，是因为线程的切换和调度的成本远远低于进程。
4. 线程的状态：NEW|RUNNABLE|BLOCKED|WAITING|TIMED_WAITING|TERMINATED

### 2.2 初始线程：线程的基本操作

#### 1. 新建线程

```java
public class NewThread {

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> System.out.println("Hello, I am t1."));
        t1.start();
    }

}
```

**注意**：不要使用run()方法开启线程，它只会在当前线程中串行执行run()方法中的代码。

#### 2. 终止线程

一般来说，线程执行完毕后就会结束，无须手动关闭。

除非你很清楚自己在做什么，否则不要使用Thread的stop方法来停止一个线程。

```java
package com.goahead.chapter02;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

public class StopThreadUnsafe {

    public static User u = new User();

    @ToString
    @Setter
    @Getter
    public static class User {

        private int id;

        private String name;

        public User() {
            this.id = 0;
            this.name = "0";
        }

    }

    public static class ChangeObjectThread extends Thread {

        @Override
        public void run() {
            while (true) {
                synchronized (u) {
                    int v = (int)(System.currentTimeMillis() / 1000);
                    u.setId(v);

                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    u.setName(String.valueOf(v));
                    Thread.yield();
                }
            }
        }
    }

    public static class ReadObjectThread extends Thread {

        @Override
        public void run() {
            while (true) {
                synchronized (u) {
                    if (u.getId() != Integer.parseInt(u.getName())) {
                        System.out.println(u.toString());
                    } else {
                        System.out.println("the same");
                    }
                }
                Thread.yield();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new ReadObjectThread().start();
        while (true) {
            Thread t = new ChangeObjectThread();
            t.start();
            Thread.sleep(150);
            t.stop();
        }
    }
}
```

#### 3. 线程中断

线程中断不会使线程立即退出，而是给线程发送一个通知，告知目标线程，有人希望你退出了。至于目标线程接到通知后如何处理，则完全由目标线程自己决定。

```java
package com.goahead.chapter02;

public class ThreadInterrupt {

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                while (true) {
                    if (Thread.currentThread().isInterrupted()) {
                        System.out.println("Interruted");
                        break;
                    }
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        System.out.println("Interruted when sleep");
                        Thread.currentThread().interrupt();
//                        e.printStackTrace();
                    }
                    Thread.yield();
                }
            }
        };

        t1.start();
        Thread.sleep(1000);
        t1.interrupt();
    }

}
```

#### 4. 等待（wait）和通知（notify）

1. 首先这两个方法不在Thread里边，而是在Object里边，任何对象都可以调用它们。
2. 当一个对象调用了wait方法后，当前线程就会在这个对象上等待。比如A线程调用了Obj.wait()，则会一直等待其他线程执行obj.notify()
3. wait方法执行后 会释放掉对象锁，而sleep方法则不会释放任何东西

```java
package com.goahead.chapter02;

public class SimpleWN {

    final static Object object = new Object();

    public static class T1 extends Thread {

        @Override
        public void run() {
            synchronized (object) {
                System.out.println(System.currentTimeMillis() + " :T1 start!");
                try {
                    System.out.println(System.currentTimeMillis() + " :T1 wait for object");
                    object.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(System.currentTimeMillis() + " :T1 End!");
            }
        }
    }

    public static class T2 extends Thread {

        @Override
        public void run() {
            synchronized (object) {
                System.out.println(System.currentTimeMillis() + " :T2 start! Notify one thread");
                object.notify();
                System.out.println(System.currentTimeMillis() + " :T2 end!");
                try {
                    Thread.sleep(20000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        Thread t1 = new T1();
        Thread t2 = new T2();
        t1.start();
        t2.start();
    }

}
```

