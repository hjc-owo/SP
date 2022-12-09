# Lab08: 多线程编程

[toc]

## 1. 实验目的

1. 掌握与线程相关的概念。
2. 掌握创建线程、挂起线程等线程相关操作。
3. 掌握线程同步的方法。

## 2. 实验内容

## 3. 实验指南

相关阅读大家感兴趣可以看一看，不看也不会影响做实验。

#### 3.1 什么是线程？

​        线程是在共享内存空间中并发的多道执行路径，它们**共享一个进程的资源**。

​        Linux线程属于用户级线程，即线程的调度是在用户空间执行的。Linux线程遵循POSIX线程接口，称为pthread。pthread在其他平台也有对应的实现，如windows。

​        线程是最小的调度单位，进程是最小的资源分配单位。

#### 3.2 并发和并行

​        并发是指在一段时间宏观上有多个程序在同时运行，但在单处理机系统中，每一时刻却仅能只有一道程序在执行，故微观上这些程序只能是分时交替执行。

​        而并行的确是多个程序同时运行。比如一个“6核6线程”的芯片，那他就支持6个线程并行（1个核心对应1个线程）。至于核心数不等于线程数的，比如“4核8线程”，大家可以了解一下**超线程技术**。

#### 3.3 线程相关



##### （1） libpthread

​        在编译使用pthread.h库的代码时，一般需要加-lpthread。

​        pthread在glibc2.34之前是在glibc里面的，之后分出来变成一个单独的库，因此有的情况下，不加-lpthread也能编译成功。[相关阅读](https://developers.redhat.com/articles/2021/12/17/why-glibc-234-removed-libpthread)

​        glibc就是stdio.h这些库的集合。[相关阅读](https://blog.csdn.net/qq_41854911/article/details/122017552)

##### （2）线程操作

阅读《Linux编程基础》8.2，有参数介绍和使用例子。

1. 创建线程

```c
int pthread_create(pthread_t _Nullable * _Nonnull __restrict,
    const pthread_attr_t * _Nullable __restrict,
    void * _Nullable (* _Nonnull)(void * _Nullable),
    void * _Nullable __restrict);
```

2. 线程退出

```c
void pthread_exit(void *ral_ptr);
```

3. 线程取消

```c
int pthread_cancel(pthread_t tid);
```

4. 线程挂起

```c
int pthread_join(pthread_t thread, void **rval_ptr);
```

5. 线程分离

```c
int pthread_detach(pthread_t tid);
```

##### （3）线程控制--同步和互斥

​          以下列举了互斥量、信号量、条件变量三种工具，三种工具的使用情况可能是重叠的，同样的问题可能有不同的解决方式。

1. 互斥量 mutex

   《Linux编程基础》8.3.1。

   使用场景举例：

   很多人去买票(ticket)，每个人是一个线程，每个人要做ticket--的操作。所以整个流程如下，

   ```
   定义mutex
   
   买票操作：
   lock mutex
   ticket <- ticket - 1
   unlock mutex
   ```

   由`lock mutex`和`unlock mutex`包起来的代码同时只有一个线程能执行。

   如果不懂`ticket--`为什么需要同时只有一个人执行，可以理解为计算机执行`ticket--`是先拿到`ticket`的当前值`ticket'`，`ticket'`减1，之后把`ticket'`赋值给`ticket`。

   ```
   ticket' <- ticket
   ticket' <- ticket' - 1
   ticket <- ticket'
   ```

   如果有两个人同时执行，可能会导致这种情况

   ```
   ticket 当前值是100
   正常流程应该是：
   person1: ticket'(1) <- ticket // ticket'(1)为100
   person1: ticket'(1) <- ticket'(1) - 1 // 99
   person1: ticket <- ticket'(1) // ticket变为99
   person2: ticket'(2) <- ticket // ticket'(2)为99
   person2: ticket'(2) <- ticket'(2) - 1 // 98
   person2: ticket <- ticket'(2) // ticket变为98
   
   如果同时执行，可能是以下情况
   person1: ticket'(1) <- ticket // ticket'(1)为100
   person2: ticket'(2) <- ticket // ticket'(2)为100
   person1: ticket'(1) <- ticket'(1) - 1 // 99
   person2: ticket'(2) <- ticket'(2) - 1 // 99
   person1: ticket <- ticket'(1) // 99
   person2: ticket <- ticket'(2) // 99
   导致一张票卖给了俩人
   
   ```

   

   mutex可以保证同时只有一个线程可以锁定这个互斥量，而且其他的线程会被挂起并不会陷入忙等待，mutex也支持了几种互斥属性，如PTHREAD_MUTEX_RECURSIVE_NP等（书P185）。

   pthread_mutex_t是一种锁，pthread还提供了pthread_spinlock_t、pthread_rwlock_t两种锁，这是[性能对比](https://blog.csdn.net/tjcwt2011/article/details/123865356)。

2. 信号量

   《Linux编程基础》8.3.2。

   信号量在之前的实验中涉及过，Linux的计数器本质上是一个整数计数器。信号量的操作无非P和V。

   P操作：

   如果信号量为0，阻塞当前线程；

   如果信号量不为0，将信号量减1。

   V操作：

   如果没有线程阻塞在这个信号量上，信号量加1；

   如果有线程阻塞在这个信号量上，唤醒其中<font color="#d d0000">1个</font>线程。

   

   信号量的使用场景举例：各种池化资源，如线程池、数据库连接池、对象池。

3. 条件变量

   《Linux编程基础》8.3.3。

   条件变量可以用来阻塞线程，也可以唤醒阻塞在这个条件变量上的一个或多个线程。条件的检测是在互斥锁的保护下进行的，因此条件变量经常和互斥量一起使用。在一些其他的语言中比如Golang，已经将互斥量作为条件变量的一个参数，条件变量的wait()方法包含了将互斥量的unlock()操作。

## 4. 实验习题

1. 用两个线程交替打印0-99，期望输出如下：

```
thread1: 0
thread2: 1
thread1: 2
thread2: 3
.
.
.
thread1: 98
thread2: 99
```

2. 修改以下代码，使counter最终值是`5000000`。

```c
#include <stdio.h>
#include <pthread.h>

#define thread_num 5

int counter;

void *add(void *arg) {
	for (int i = 0; i < 1000000; i++) {
		++counter;
	}
}

int main() {
	pthread_t tids[thread_num];
	for (int i = 0; i < thread_num; i++) {
		pthread_create(&tids[i], NULL, add, NULL);
	}
	for (int i = 0; i < thread_num; i++) {
		pthread_join(tids[i], NULL);
	}
	printf("counter=%d\n", counter);
}
```

3. 用C语言完成[Leetcode 1117 H2O 生成](https://leetcode.cn/problems/building-h2o/)，给出通过的代码。

4. 消费者生产者模型是条件变量最经典的使用场景之一，该问题描述了共享固定大小缓冲区的两个线程——即所谓的“生产者”和“消费者”——在实际运行时会发生的问题。生产者的主要作用是生成一定量的数据放到缓冲区中，然后重复此过程。与此同时，消费者也在缓冲区消耗这些数据。该问题的关键就是要保证生产者不会在缓冲区满时加入数据，消费者也不会在缓冲区中空时消耗数据。

   生产者消费者问题主要要注意以下三点：

   - 在缓冲区为空时，消费者不能再进行消费
   - 在缓冲区为满时，生产者不能再进行生产
   - 在一个线程进行生产或消费时，其余线程不能再进行生产或消费等操作，即保持线程间的同步

   假设缓冲区上限为 20，生产者和消费者线程各 10 个，请编写程序实现一个生产者消费者模型。**在每次生产、消费时将当前动作类型（produce/consume）与缓冲区内容量输出到屏幕，给出代码和操作步骤。**
