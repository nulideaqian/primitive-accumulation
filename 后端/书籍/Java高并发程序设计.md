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

#### 5. 挂起（suspend）和继续执行（resume）线程

1. 被挂起的线程必须要等到resume方法操作后，才能继续执行。
2. suspend方法执行后，线程暂停的同时，并不会释放任何锁资源。必须等到该线程执行了resume操作后才可以继续运行，万一resume方法在suspend方法之前执行，那么挂起的线程可能很难有机会继续执行，它占用的锁永远不会释放，系统会有问题。

```java
package com.goahead.chapter02;

public class BadSuspend {

    public static Object u = new Object();

    static ChangeObjectThread t1 = new ChangeObjectThread("t1");
    static ChangeObjectThread t2 = new ChangeObjectThread("t2");

    static class ChangeObjectThread extends Thread {

        public ChangeObjectThread(String name) {
            super.setName(name);
        }

        @Override
        public void run() {
            synchronized (u) {
                System.out.println("in " + getName());
                Thread.currentThread().suspend();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        t1.start();
        Thread.sleep(100);
        t2.start();
        t1.resume();
        // 如果此处使用Thread.sleep 使得线程 暂停  从而使 子线程先 执行  则不会有问题
        t2.resume();
        // join 让主线程等待子线程结束后 再执行
        // 通过jstack 可以查看 运行的线程
        t1.join();
        t2.join();
    }

}
```

#### 6. 等待线程结束（join）和谦让（yield）

1. 很多时候一个线程的输入依赖其他一个or多个线程的输入，此时线程需要等待依赖的线程执行完毕后才开始执行。比如SE和开发 一般要等SA调研清楚需求后开工。

2. JDK提供了join的机制，第一个无限制等待，第二个限制时间，超时了就不等了

   ```java
   public final void join() throws InterruptedException;
   public final synchronized void join(long millis) throws InterruptedException;
   ```

3. 举个例子

   ```java
   package com.goahead.chapter02;
   
   public class JoinMain {
   
       public volatile static int i = 0;
   
       public static class AddThread extends Thread {
   
           @Override
           public void run() {
               for (i = 0; i < 10000000; i++);
           }
       }
   
       public static void main(String[] args) throws InterruptedException {
           AddThread at = new AddThread();
           at.start();
   //        at.join();
           System.out.println(i);
       }
   
   }
   ```

4. yield是一个静态方法，一旦执行，则当前线程会让出CPU，重新参与CPU竞争。

### 2.3 volatile与Java内存模型（JMM）

1. Java使用了一些特殊的操作或者关键字来声明，告诉虚拟机，在这个地方要尤其注意，不能随意变动优化目标指令，其中volatile就是其中之一。

2. volatile能保证数据的可见性和有序性，但是不能保证原子性。

3. 举个例子

   ```java
   package com.goahead.chapter02;
   
   public class NoVisibility {
   
       private volatile static boolean ready;
   
       private static int number;
   
       private static class ReaderThread extends Thread {
   
           @Override
           public void run() {
               while (!ready);
               System.out.println(number);
           }
   
       }
   
       public static void main(String[] args) throws InterruptedException {
           new ReaderThread().start();
           Thread.sleep(1000);
           ready = true;
           number = 44;
           Thread.sleep(10000);
       }
   
   }
   ```

### 2.4 分门别类的管理：线程组

1. 如果线程数量众多，但是职责明确，我们可以将相同功能的线程放到一个线程组里边。

2. 举个栗子：

   ```java
   package com.goahead.chapter02;
   
   public class ThreadGroupName implements Runnable {
   
       public static void main(String[] args) {
           ThreadGroup tg = new ThreadGroup("PrintGroup");
           Thread t1 = new Thread(tg, new ThreadGroupName(), "T1");
           Thread t2 = new Thread(tg, new ThreadGroupName(), "T2");
           t1.start();
           t2.start();
           System.out.println(tg.activeCount());
           tg.list();
       }
   
       @Override
       public void run() {
           String groupAndName = Thread.currentThread().getThreadGroup().getName() + "-" + Thread.currentThread().getName();
           while (true) {
               System.out.println("I am " + groupAndName);
               try {
                   Thread.sleep(3000);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
           }
       }
   }
   ```

3. 作者建议大家在创建线程和线程组的时候，给它们取一个好听的名字，定位问题的时候方便，不然就是一堆：Thread-0、Thread-1等等

### 2.5 驻守后台：守护线程（Daemon）

1. 守护线程实在后台默默运行的线程，比如垃圾回收，JIT线程。
2. 与守护线程相对的就是用户线程，也可以称为系统的工作线程
3. 当Java应用内只剩下守护线程时候，Java虚拟机会自然的退出

```java
package com.goahead.chapter02;

public class DaemonDemo {

    public static class DaemonT extends Thread {
        @Override
        public void run() {
            while (true) {
                System.out.println("I am alive");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t = new DaemonT();
        // 如果不设置为守护线程，则主线程结束以后，依然在执行
//        t.setDaemon(true);

        t.start();

        Thread.sleep(2000);
    }

}
```

### 2.6 先做重要的事：线程优先级

1. 优先级越高的线程在竞争资源的时候更有优势，更加可能抢到资源。
2. 低优先级的线程 有可能会发生饥饿现象，所以在严格的场合，还是需要自己在应用层解决线程调度的问题。

```java
package com.goahead.chapter02;

public class PriorityDemo {

    public static class HightPriority extends Thread {
        static int count = 0;

        @Override
        public void run() {
            while (true) {
                synchronized (PriorityDemo.class) {
                    count++;
                    if (count > 10000000) {
                        System.out.println("HightPriority is complete");
                        break;
                    }
                }
            }
        }
    }

    public static class LowPriority extends Thread {
        static int count = 0;

        @Override
        public void run() {
            while (true) {
                synchronized (PriorityDemo.class) {
                    count++;
                    if (count > 10000000) {
                        System.out.println("LowPriority is complete");
                        break;
                    }
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 100; i++) {
            HightPriority high = new HightPriority();
            LowPriority low = new LowPriority();
            high.setPriority(Thread.MAX_PRIORITY);
            low.setPriority(Thread.MIN_PRIORITY);
            high.start();
            low.start();
            Thread.sleep(100);
            System.out.println(" ========================= ");
        }
    }

}
```

### 2.7 线程安全的概念与关键字synchronized

1. 并行程序设计一大关注重点就是线程安全。
2. volatile关键字只能保证数据的可见性
3. synchronized关键字是实现线程间的同步，它的工作是对同步的代码块加锁，确保一次只有一个线程进入同步块，它还能保证线程间的可见性和有序性。
4. 可以作用于：对象，实例方法和静态方法

### 2.8 程序中的幽灵：隐蔽的错误

#### 1. 并发下的ArrayList

多个线程对ArrayList操作，可能导致如下三种情况：

1. 结果正常
2. 数组越界异常：由于没有锁的保护，另外一个线程访问到不一致的内部状态，导致越界
3. arraylist的重复写入，结果看着不正常

```java
package com.goahead.chapter02;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ArrayListMultiThread {

    static List<Integer> al = Collections.synchronizedList(new ArrayList<>());

    public static class AddThread implements Runnable {

        @Override
        public void run() {
            for (int i = 0; i < 1000000; i++) {
                al.add(i);
            }
        }

    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new AddThread());
        Thread t2 = new Thread(new AddThread());
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(al.size());
    }

}
```

解决方案：

1. 使用Vector代替ArrayList
2. 使用Collections.synchronizedList(new ArrayList<>());

#### 2. 并发下诡异的HashMap

1. HashMap本身是性能不安全的
2. JDK7 线程冲突下的HashMap有可能出现环链，死循环

3. 并发场景下推荐使用：ConcurrentHashMap

#### 3. 初学者常见的问题：错误的加锁

如下代码，有问题：

```java
package com.goahead.chapter02;

public class BadLockOnInteger implements Runnable {

    public static Integer i = 0;

    static BadLockOnInteger instance = new BadLockOnInteger();

    @Override
    public void run() {
        for (int j = 0; j < 120; j++) {
            synchronized (i) {
                i++;
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(i);
    }
}
```

如上代码有个冷知识：Integer默认-128到127是固定的对象，IntegerCache.cache数组存放，-128是默认的，127可以通过虚拟机参数更改-Djava.lang.Integer.IntegerCache.high=10000

## 第3章 JDK并发包

> JDK为了更好的支持并发程序，提供了大量实用的API和框架

### 3.1 多线程的团队协作：同步控制

> synchronized, wait, notify完成了对临界区的访问控制，它们的增强版是：重入锁

#### 1. 关键字synchronized的功能扩展：重入锁

> 重入锁可以完全替代关键字synchronized，JDK5.0之前冲入锁的性能要比synchronized好很多

```java
package com.goahead.chapter02;

import java.util.concurrent.locks.ReentrantLock;

public class ReenterLock implements Runnable {

    public static ReentrantLock lock = new ReentrantLock();

    public static int i = 0;

    @Override
    public void run() {
        for (int j = 0; j < 10000000; j++) {
            lock.lock();
            try {
                i++;
            } finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ReenterLock reenterLock = new ReenterLock();
        Thread t1 = new Thread(reenterLock);
        Thread t2 = new Thread(reenterLock);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(i);
    }

}
```

重入锁的特点：

- 显示的操作过程，开发人员必须手动指定何时加锁，何时释放锁。

- 注意：必须在退出临界区时，释放掉自己持有的锁。

- 为何叫重入锁（Re-Entrant-Lock）？因为这个锁是可以反复进入的，比如下面的代码：

  ```java
  lock.lock();
  lock.lock();
  try {
  	i++;
  } finally {
  	lock.unlock();
      lock.unlock();
  }
  ```

重入锁提供了中断处理能力

1. 中断响应

   synchronized关键字，如果一个线程在等待锁，它只有两种情况发生：要么获得锁继续执行，要么一直等待锁。相对于synchronized，重入锁提供了一个机制：等待锁的过程是可以中断的，即某一线程在等待锁的过程中，程序可以根据需要取消对锁的请求。

   ```java
   package com.goahead.chapter03;
   
   import java.util.concurrent.locks.ReentrantLock;
   
   public class IntLock implements Runnable {
   
       public static ReentrantLock lock1 = new ReentrantLock();
       public static ReentrantLock lock2 = new ReentrantLock();
   
       int lock;
   
       public IntLock(int lock) {
           this.lock = lock;
       }
   
       @Override
       public void run() {
           try {
               if (lock == 1) {
                   lock1.lockInterruptibly();
                   try {
                       Thread.sleep(500);
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   }
                   lock2.lockInterruptibly();
               } else {
                   lock2.lockInterruptibly();
                   try {
                       Thread.sleep(500);
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   }
                   lock1.lockInterruptibly();
               }
           } catch (InterruptedException e) {
               e.printStackTrace();
           } finally {
               if (lock1.isHeldByCurrentThread()) {
                   lock1.unlock();
               }
               if (lock2.isHeldByCurrentThread()) {
                   lock2.unlock();
               }
               System.out.println(Thread.currentThread().getId() + ": 线程退出");
           }
       }
   
       public static void main(String[] args) throws InterruptedException {
           IntLock r1 = new IntLock(1);
           IntLock r2 = new IntLock(2);
           Thread t1 = new Thread(r1);
           Thread t2 = new Thread(r2);
           t1.start();
           t2.start();
           Thread.sleep(1000);
           t1.interrupt();
       }
   }
   ```

2. 锁申请等待限时

   > 通常我们无法判断为什么线程一直获取不到锁，有可能是死锁了，也有可能饥饿了。如果给定一个等待时间，让线程自动放弃，对系统来说是有意义的。

   使用tryLock方法实现锁等待超时：

   ```java
   package com.goahead.chapter03;
   
   import java.util.concurrent.TimeUnit;
   import java.util.concurrent.locks.ReentrantLock;
   
   public class TimeLock implements Runnable {
   
       public static ReentrantLock lock = new ReentrantLock();
   
       @Override
       public void run() {
           try {
               if (lock.tryLock(5, TimeUnit.SECONDS)) {
                   System.out.println(Thread.currentThread().getName() + "获取锁成功");
                   Thread.sleep(6000);
               } else {
                   System.out.println(Thread.currentThread().getName() + "获取锁失败");
               }
           } catch (InterruptedException e) {
               e.printStackTrace();
           } finally {
               if (lock.isHeldByCurrentThread()) {
                   lock.unlock();
               }
           }
       }
   
       public static void main(String[] args) {
           Thread t1 = new Thread(new TimeLock(), "t1");
           Thread t2 = new Thread(new TimeLock(), "t2");
           t1.start();
           t2.start();
       }
   }
   ```

3. 公平锁

   - 大部分情况下，锁都是不公平的。
   - 公平锁可以避免饥饿的情况发生。
   - 公平锁由于系统需要维护一个队列，所以性能非常低。如果没有什么特殊要求，则不要使用公平锁。

   ```java
   package com.goahead.chapter03;
   
   import java.util.concurrent.locks.ReentrantLock;
   
   public class FairLock implements Runnable {
   
   //    public static ReentrantLock fairLock = new ReentrantLock(true);
       public static ReentrantLock fairLock = new ReentrantLock(false);
   
       @Override
       public void run() {
           while (true) {
               fairLock.lock();
               System.out.println(Thread.currentThread().getName() + " 获取到锁");
               fairLock.unlock();
           }
       }
   
       public static void main(String[] args) {
           FairLock item = new FairLock();
           Thread t1 = new Thread(item, "t1");
           Thread t2 = new Thread(item, "t2");
           t1.start();
           t2.start();
       }
   }
   ```

**总结**：

重入锁有如下的**函数：**

| 函数名            | 作用                               |
| ----------------- | ---------------------------------- |
| lock              | 加锁，等待                         |
| lockInterruptibly | 加锁，优先响应中断事件             |
| tryLock-无参      | 获取锁，获取不到则放弃，获取到继续 |
| tryLock-有参      | 获取锁，如果超时，则放弃           |
| unlock            | 释放锁                             |

重入锁的**实现：**

1. **原子状态：**使用CAS来存储当前锁的状态，判断锁是否已经被别的线程持有了。
2. **等待队列：**所有没有请求到锁的线程，会进入等待队列进行等待。待有线程释放锁后，系统就能从等待队列中唤醒一个线程，继续工作。
3. **阻塞原语**park和unpark，用来挂起和恢复线程。

#### 2. 重入锁的好搭档：condition

> condition和wait/notify的原理类似，不过wait和notify是配合关键字synchronized使用的。

如下简单演示了Condition的功能：

```java
package com.goahead.chapter03;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class ReenterLockCondition implements Runnable {

    public static ReentrantLock lock = new ReentrantLock();

    public static Condition condition = lock.newCondition();

    @Override
    public void run() {
        try {
            lock.lock();
            // 调用后，该线程将释放掉自己的锁
            System.out.println(Thread.currentThread().getName() + ": Thread is going on");
            condition.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            condition.signal();
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ReenterLockCondition reenterLockCondition = new ReenterLockCondition();
        Thread t1 = new Thread(reenterLockCondition, "t1");
        Thread t2 = new Thread(reenterLockCondition, "t2");
        t1.start();
        t2.start();
        System.out.println("主线程停顿2秒哈");
        Thread.sleep(2000);
        lock.lock();
        // 系统会从当前Condition对象的等待队列中唤醒一个线程
        condition.signal();
        lock.unlock();
    }
}
```

#### 3. 允许多个线程同时访问：信号量（Semaphore）

> 信号量是对锁的扩展。无论是内部锁synchronized还是重入锁reentrantLock，一次都只允许一个线程访问一个资源，而信号量可以访问多个。

举个semaphore的小栗子：

```java
package com.goahead.chapter03;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class SemaphoreDemo implements Runnable {

    final Semaphore semp = new Semaphore(5);

    @Override
    public void run() {
        try {
            semp.acquire();
            Thread.sleep(2000);
            System.out.println(Thread.currentThread().getName() + ": done!");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            semp.release();
        }
    }

    public static void main(String[] args) {
        ExecutorService exec = Executors.newFixedThreadPool(20);
        final SemaphoreDemo demo = new SemaphoreDemo();
        for (int i = 0; i < 20; i++) {
            exec.submit(demo);
        }
    }

}
```

#### 4. ReadWriteLock读写锁

> 读写分离锁可以有效地帮助减少锁竞争，提升性能。

重入锁和内部锁在任何情况下，一个线程都会阻塞其他线程的执行。而我们知道所有的读线程在一起工作的时候并不会破坏数据的完整性，其实是可以并行执行的。

从上边的表述来看：如果一个系统读操作的次数远远大于写的操作次数，使用读写锁可以有效的提升系统的性能。

举个栗子：

```java
package com.goahead.chapter03;

import java.time.Instant;
import java.util.Random;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteLockDemo {

    private static Lock lock = new ReentrantLock();

    private static ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();

    private static Lock readLock = reentrantReadWriteLock.readLock();

    private static Lock writeLock = reentrantReadWriteLock.writeLock();

    private int value;

    public Object handleRead(Lock lock) throws InterruptedException {
        try {
            lock.lock();
            Thread.sleep(1000);
            return value;
        } finally {
            lock.unlock();
        }
    }

    public void handleWrite(Lock lock, int index) throws InterruptedException {
        try {
            lock.lock();
            Thread.sleep(1000);
            value = index;
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        long startTime = Instant.now().toEpochMilli();
        final ReadWriteLockDemo demo = new ReadWriteLockDemo();
        Runnable readRunnable = new Runnable() {
            @Override
            public void run() {
                try {
//                    demo.handleRead(readLock);
                    demo.handleRead(lock);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        Runnable writeRunnable = new Runnable() {
            @Override
            public void run() {
                try {
//                    demo.handleWrite(writeLock, new Random().nextInt());
                    demo.handleWrite(lock, new Random().nextInt());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };

        for (int i = 0; i < 18; i++) {
            new Thread(readRunnable).start();
        }
        for (int i = 18; i < 20; i++) {
            new Thread(writeRunnable).start();
        }
        long endTime = Instant.now().toEpochMilli();
        System.out.println(endTime - startTime);
    }

}
```

#### 5. 倒计数器：CountDownLatch

它常用与控制线程等待，即主线程等待倒数计数器结束后，再开始执行。

```java
package com.goahead.chapter03;

import java.util.Random;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CountDownLatchDemo implements Runnable {

    static final CountDownLatch end = new CountDownLatch(10);

    static final CountDownLatchDemo demo = new CountDownLatchDemo();

    @Override
    public void run() {
        try {
            Thread.sleep(new Random().nextInt(10)*1000);
            System.out.println("Check complete");
            end.countDown();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService exec = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 10; i++) {
            exec.submit(demo);
        }
        end.await();
        System.out.println("Fire");
        exec.shutdown();
    }

}
```

#### 6. 循环栅栏

举个栗子：

```java
package com.goahead.chapter03;

import java.util.Random;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierDemo {

    public static class Solider implements Runnable {

        private String solider;

        private final CyclicBarrier cyclic;

        public Solider(String solider, CyclicBarrier cyclic) {
            this.solider = solider;
            this.cyclic = cyclic;
        }

        @Override
        public void run() {
            try {
                cyclic.await();
                doWork();
                cyclic.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }

        void doWork() {
            try {
                Thread.sleep(Math.abs(new Random().nextInt()%10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(solider + " 任务完成");
        }
    }

    public static class BarrierRun implements Runnable {

        boolean flag;

        int N;

        public BarrierRun(boolean flag, int n) {
            this.flag = flag;
            N = n;
        }

        @Override
        public void run() {
            if (flag) {
                System.out.println("司令：【士兵" + N + "个，完成任务啦啦啦啦");
            } else {
                System.out.println("司令：【士兵" + N + "个，集合完毕");
                flag = true;
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        final int N = 10;
        Thread[] allSoldier = new Thread[N];
        boolean flag = false;
        CyclicBarrier cyclic = new CyclicBarrier(N*2, new BarrierRun(flag, N));
        System.out.println("集合队伍");
        for (int i = 0; i < N; i++) {
            System.out.println("士兵 " + i + " 报道!");
            allSoldier[i] = new Thread(new Solider("士兵" + i, cyclic));
            allSoldier[i].start();
            Thread.sleep(200);
        }
    }

}
```

关于InterruptedException异常：在线程的等待过程中，线程被中断了。

#### 7. 线程阻塞工具类：LockSupport

作用：它可以在线程任何位置让线程阻塞

LockSupport的静态方法park()可以阻塞当前线程。

### 3.2 线程复用：线程池

> 线程如果不加以控制和管理，反而会对性能产生不利的影响
>
> 当线程数量太多的时候，有可能会耗尽CPU和内存资源
>
> 1. 虽然线程是轻量级的工具，但是其本身创建和关闭需要花费很长时间
> 2. 线程本身也是要消耗内存空间的，大量的线程会抢占宝贵的资源，如果处理不当，会造成OOM的异常。
>
> 所以对线程的使用需要掌握一个度

#### 1. 什么是线程池

使用线程池，创建线程变成了从线程池中获取空闲线程，关闭线程变成了向线程池中归还线程。

#### 2. 不要重复发明轮子：JDK对线程的支持

> JDK提供了一整套Executor框架
>
> Executor，ExecutorService，AbstractExecutorService，ThreadPoolExecutor，Executors(工厂)
>
> Executor框架提供了两大类的线程池：ExecutorService和ScheduleExecutorService

1. 固定大小的线程池

   ```java
   package com.goahead.chapter03;
   
   import java.util.concurrent.ExecutorService;
   import java.util.concurrent.Executors;
   
   public class ThreadPoolDemo {
   
       public static class MyTask implements Runnable {
   
           @Override
           public void run() {
               System.out.println(System.currentTimeMillis() + ": Thread ID: " + Thread.currentThread().getId());
               try {
                   Thread.sleep(1000);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
           }
       }
   
       public static void main(String[] args) {
           MyTask myTask = new MyTask();
   //        ExecutorService es = Executors.newFixedThreadPool(5);
           ExecutorService es = Executors.newCachedThreadPool();
           for (int i = 0; i < 10; i++) {
               es.submit(myTask);
               try {
                   Thread.sleep(500);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
           }
       }
   
   }
   ```

   ```java
   package com.goahead.chapter03;
   
   import java.time.LocalDateTime;
   import java.util.concurrent.Executors;
   import java.util.concurrent.ScheduledExecutorService;
   import java.util.concurrent.TimeUnit;
   
   public class ScheduledExecutorServiceDemo {
   
       public static void main(String[] args) {
           ScheduledExecutorService ses = Executors.newScheduledThreadPool(10);
           ses.scheduleAtFixedRate(new Runnable() {
               @Override
               public void run() {
                   try {
                       Thread.sleep(1000);
                       LocalDateTime now = LocalDateTime.now();
                       System.out.println(now.getHour() + ": " + now.getMinute() + ": " + now.getSecond());
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   }
               }
           }, 0, 2, TimeUnit.SECONDS);
       }
   
   }
   ```

   ps:

   1. 如果任务遇到异常，那么后续的所有子任务都会停止调度。
   2. 周期如果太短，那么任务就会在上一个任务结束后立即被调用。

#### 3. 刨根究底：核心线程池的内部实现

1. newFixedThreadPool，newSingleThreadPool和newCachedThreadPool都是ThreadPoolExecutor类的具体表现。
2. 理解线程池的核心参数：核心线程数，最大线程数，空闲线程存活时间，任务队列，拒绝策略

#### 4. 超负载了怎么办：拒绝策略

代码示例:

```java
package com.goahead.chapter03;

import java.util.concurrent.*;

public class RejectThreadPoolDemo {

    public static class MyTask implements Runnable {

        @Override
        public void run() {
            System.out.println(System.currentTimeMillis() + ": Thread ID: " + Thread.currentThread().getId());
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }

    public static void main(String[] args) throws InterruptedException {
        MyTask task = new MyTask();
        ExecutorService es = new ThreadPoolExecutor(5, 5, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingDeque<>(10),
                Executors.defaultThreadFactory(),
                (r, executor) -> System.out.println(r.toString() + " is discard"));
        for (int i = 0; i < Integer.MAX_VALUE; i++) {
            es.submit(task);
            Thread.sleep(10);
        }
    }

}
```

#### 5. 自定义线程创建：ThreadFactory

1. ThreadFactory是一个接口，它只有一个创建线程的方法：Thread newThread(Runnable r);

2. 使用自定义线程，我们可以很好的跟踪线程的创建，命名和优先级等信息；使得我们更加自由的控制线程池中所有线程的状态。比如我们可以将创建的线程自定义为守护线程Deamon

#### 6. 我的应用我做主：扩展线程池

1. 在实际应用中，可以对其进行扩展来实现对线程池运行状态的跟踪，输出一些有用的调试信息，以帮助信息诊断，对于错误排查是非常有用的。

举个栗子：

```java
package com.goahead.chapter03;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class ExtThreadPool {

    public static class MyTask implements Runnable {

        public String name;

        public MyTask(String name) {
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println("正在执行: " + ": Thread ID: " + Thread.currentThread().getId() + ", Task Name = " + name);
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService es = new ThreadPoolExecutor(5, 5, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingDeque<Runnable>()) {
            @Override
            protected void beforeExecute(Thread t, Runnable r) {
                System.out.println("准备执行: " + ((MyTask)r).name);
            }

            @Override
            protected void afterExecute(Runnable r, Throwable t) {
                System.out.println("执行完成: " + ((MyTask)r).name);
            }

            @Override
            protected void terminated() {
                System.out.println("线城池退出");
            }
        };
        for (int i = 0; i < 10; i++) {
            MyTask task = new MyTask("TASK-GEYM-" + i);
            es.execute(task);
            Thread.sleep(10);
        }
        es.shutdown();
    }

}
```

#### 7. 合理的选择：优化线程池线程数量

1. 线程池的大小对系统的性能有着一定的影响。
2. 只要避免极大和极小极端情况，线程池的大小对系统的影响并不会太大。

#### 8. 堆栈去哪里了：在线程池中寻找堆栈

两种方式提交任务可以打印任务的堆栈异常：

- 使用execute方法
- 或者将submit的结果返回，调用其get方法

为了打印业务日志上下文，自定义线程池例子：

```java
package com.goahead.chapter03;

import java.util.concurrent.*;

public class TraceThreadPoolExecutor extends ThreadPoolExecutor {

    public TraceThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }

    @Override
    public void execute(Runnable command) {
        super.execute(wrap(command, clientTrace(), Thread.currentThread().getName()));
    }

    @Override
    public Future<?> submit(Runnable task) {
        return super.submit(wrap(task, clientTrace(), Thread.currentThread().getName()));
    }

    private Exception clientTrace() {
        return new Exception("Client stack trace");
    }

    private Runnable wrap(final Runnable task, final Exception clientStack, String clientThreadName) {
        return new Runnable() {
            @Override
            public void run() {
                try {
                    task.run();
                } catch (Exception e) {
                    clientStack.printStackTrace();
                    throw e;
                }
            }
        };
    }
}
```

#### 9. 分而治之：Fork/Join框架

1. 当一个线程企图帮助另外一个线程的时候，往往从任务队列底部开始获取数据，而线程自己本身从顶部获取数据，因此这么操作有利于避免数据竞争。

2. 你可以向ForkJoinPool线程池提交一个ForkJoinTask任务。所谓的ForkJoinTask任务就是支持fork()方法分解以及join方法等待的任务。

   举个例子：

   ```java
   package com.goahead.chapter03;
   
   import java.util.ArrayList;
   import java.util.concurrent.ExecutionException;
   import java.util.concurrent.ForkJoinPool;
   import java.util.concurrent.ForkJoinTask;
   import java.util.concurrent.RecursiveTask;
   
   public class CountTask extends RecursiveTask<Long> {
   
       private static final int THRESHOLD = 10000;
   
       private long start;
   
       private long end;
   
       public CountTask(long start, long end) {
           this.start = start;
           this.end = end;
       }
   
       public Long compute() {
           long sum = 0;
           boolean canCompute = (end - start) < THRESHOLD;
           if (canCompute) {
               for (long i = start; i <= end; i++) {
                   sum += i;
               }
           } else {
               long step = (start + end) / 100;
               ArrayList<CountTask> subTasks = new ArrayList<>();
               long pos = start;
               for (int i = 0; i < 100; i++) {
                   long lastOne = pos + step;
                   if (lastOne > end) {
                       lastOne = end;
                   }
                   CountTask subTask = new CountTask(pos, lastOne);
                   pos += step + 1;
                   subTasks.add(subTask);
                   subTask.fork();
               }
               for (CountTask t: subTasks) {
                   // 此处join方法和Thread的不同，它是ForkJoinTask中的方法
                   sum += t.join();
               }
           }
           return sum;
       }
   
       public static void main(String[] args) {
           ForkJoinPool forkJoinPool = new ForkJoinPool();
           CountTask task = new CountTask(0, 200000L);
           ForkJoinTask<Long> result = forkJoinPool.submit(task);
           try {
               Long res = result.get();
               System.out.println("sum = " + res);
           } catch (InterruptedException e) {
               e.printStackTrace();
           } catch (ExecutionException e) {
               e.printStackTrace();
           }
       }
   
   }
   ```

   - 使用fork()方法提交子任务
   - 任务不能划分的层次很多，有可能创建很多线程，导致性能严重下降；
   - 函数调用栈增多，最终导致栈溢出

#### 10. Guava中对线程池的扩展

1. 特殊的DirectExecutor线程池

   它并没有真的创建或使用额外线程，他总是将任务在当前线程中直接执行。

   软件从设计的角度来说，抽象是软件设计的根本和精髓。我们总是希望并且倾向于使用通用代码来处理不同的场景，因此需要对不同的场景进行他统一的建模和抽象。

   ```java
   public static void main(String[] args) throws InterruptedException {
           Executor executor = MoreExecutors.directExecutor();
           executor.execute(() -> System.out.println("I am running in " + Thread.currentThread().getName()));
       }
   ```

2. Deamon线程池

   使用MoreExecutors.getExitingExecutorService(executor);可以将普通线程池转化为Daemon线程池的方法。

   ```java
       public static void main(String[] args) throws InterruptedException {
           ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newFixedThreadPool(2);
           // 如果没有下面这行，因线程池的存在，应用无法正常结束
   //        MoreExecutors.getExitingExecutorService(executor);
           executor.execute(() -> System.out.println("I am running in " + Thread.currentThread().getName()));
       }
   ```

### 3.3 不要重复发明轮子：JDK的并发容器

#### 1. 超好用的工具类：并发集合简介

| 名称                  | 值                  | 说明                   |
| --------------------- | ------------------- | ---------------------- |
| ConcurrentHashMap     | 高效的并发HashMap   |                        |
| CopyOnWriteArrayList  | 线程安全的ArrayList | 适合读多写少的场景     |
| ConcurrentLinkedQueue | 高效的并发队列      | 使用链表实现           |
| BlockingQueue         | 阻塞队列            | 适合作为数据共享的通道 |
| ConcurrentSkipListMap | 跳表的实现          |                        |

#### 2. 线程安全的HashMap

两种方式：

1. Collections.synchronizedMap(new HashMap<>()); -- 性能不是太好，建议使用第二种
2. ConcurrentHashMap