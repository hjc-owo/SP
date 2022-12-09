# Lab08 - Assignment

>姓名：胡峻诚
>
>学号：19241027

## 1. 用两个线程交替打印0-99，期望输出如下：

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

```c
#include <pthread.h>
#include <stdio.h>

int n = 0;
pthread_mutex_t mutex;

void* print_even(void* arg) {
    while (n < 99) {
        pthread_mutex_lock(&mutex);
        if (n % 2 == 0) {
            printf("thread1: %d\n", n);
            n++;
        }
        pthread_mutex_unlock(&mutex);
    }
    pthread_exit(NULL);
}

void* print_odd(void* arg) {
    while (n < 99) {
        pthread_mutex_lock(&mutex);
        if (n % 2 != 0) {
            printf("thread2: %d\n", n);
            n++;
        }
        pthread_mutex_unlock(&mutex);
    }
    pthread_exit(NULL);
}

int main() {
    pthread_t thread1, thread2;

    pthread_mutex_init(&mutex, NULL);

    pthread_create(&thread1, NULL, print_even, NULL);
    pthread_create(&thread2, NULL, print_odd, NULL);

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    pthread_mutex_destroy(&mutex);

    return 0;
}
```

![截屏2022-12-08 23.33.13](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202212082333749.png)

## 2. 修改以下代码，使counter最终值是`5000000`。

```c
#include <stdio.h>
#include <pthread.h>

#define thread_num 5

int counter;
pthread_mutex_t mutex;

void *add(void *arg) {
	for (int i = 0; i < 1000000; i++) {
		pthread_mutex_lock(&mutex);
		++counter;
		pthread_mutex_unlock(&mutex);
	}
}

int main() {
	pthread_mutex_init(&mutex, NULL);

	pthread_t tids[thread_num];
	for (int i = 0; i < thread_num; i++) {
		pthread_create(&tids[i], NULL, add, NULL);
	}
	for (int i = 0; i < thread_num; i++) {
		pthread_join(tids[i], NULL);
	}

	pthread_mutex_destroy(&mutex);

	printf("counter=%d\n", counter);
}
```

![截屏2022-12-08 23.58.05](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202212082358041.png)

## 3. 用C语言完成[Leetcode 1117 H2O 生成](https://leetcode.cn/problems/building-h2o/)，给出通过的代码。

```c
typedef struct {
    // User defined data may be declared here.
    pthread_mutex_t m;
    pthread_cond_t cv;
    int h, o;
} H2O;

H2O* h2oCreate() {
    H2O* obj = (H2O*) malloc(sizeof(H2O));
    // Initialize user defined data here.
    pthread_mutex_init(&obj->m, NULL);
    pthread_cond_init(&obj->cv, NULL);
    obj->h = obj->o = 0;
    return obj;
}

void reset(H2O* obj) {
    if (obj->h == 2 && obj->o == 1)
        obj->h = obj->o = 0;
}

void hydrogen(H2O* obj) {
    pthread_mutex_lock(&obj->m);
    while (obj->h >= 2) {
        pthread_cond_wait(&obj->cv, &obj->m);
        usleep(300);
    }
    // releaseHydrogen() outputs "H". Do not change or remove this line.
    releaseHydrogen();
    obj->h++;
    reset(obj);
    pthread_mutex_unlock(&obj->m);
    pthread_cond_broadcast(&obj->cv);
}

void oxygen(H2O* obj) {
    pthread_mutex_lock(&obj->m);
    while (obj->o >= 1) {
        pthread_cond_wait(&obj->cv, &obj->m);
        usleep(300);
    }
    // releaseOxygen() outputs "O". Do not change or remove this line.
    releaseOxygen();
    obj->o++;
    reset(obj);
    pthread_mutex_unlock(&obj->m);
    pthread_cond_broadcast(&obj->cv); 
}

void h2oFree(H2O* obj) {
    // User defined data may be cleaned up here.
    pthread_mutex_destroy(&obj->m);
    pthread_cond_destroy(&obj->cv);
    free(obj);
}
```

![截屏2022-12-09 12.05.13](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202212091205448.png)

晕了，为啥中间要`usleep()`歇歇才能跑对。

## 4. 生产者消费者模型

生产者消费者模型是条件变量最经典的使用场景之一，该问题描述了共享固定大小缓冲区的两个线程——即所谓的“生产者”和“消费者”——在实际运行时会发生的问题。生产者的主要作用是生成一定量的数据放到缓冲区中，然后重复此过程。与此同时，消费者也在缓冲区消耗这些数据。该问题的关键就是要保证生产者不会在缓冲区满时加入数据，消费者也不会在缓冲区中空时消耗数据。

生产者消费者问题主要要注意以下三点：

- 在缓冲区为空时，消费者不能再进行消费
- 在缓冲区为满时，生产者不能再进行生产
- 在一个线程进行生产或消费时，其余线程不能再进行生产或消费等操作，即保持线程间的同步

假设缓冲区上限为 20，生产者和消费者线程各 10 个，请编写程序实现一个生产者消费者模型。**在每次生产、消费时将当前动作类型（produce/consume）与缓冲区内容量输出到屏幕，给出代码和操作步骤。**

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define MAX (50)

pthread_cond_t consumer_cv; // 条件变量
pthread_cond_t producer_cv;
pthread_mutex_t mutex; // 互斥锁

int consumeStop = 0;
int produceStop = 0;

int now = 0;

// 缓冲区结构体
struct {
    int capacity; // 缓冲区容量
    int *data;
    int product_num; // 缓冲区目前的元素个数
    int produce_pos;
    int consume_pos;
} buffer;

// 生产者线程函数
void *produce() {
    while (1) {
        pthread_mutex_lock(&mutex);
        while (buffer.product_num >= buffer.capacity && !produceStop)
            pthread_cond_wait(&producer_cv, &mutex);
        if (produceStop) {
            pthread_mutex_unlock(&mutex);
            break;
        }
        int item = now++;
        buffer.data[buffer.produce_pos] = item;
        printf("Produce: %d at position %d, buffer contains %d product now.\n", item, buffer.produce_pos, ++buffer.product_num);
        fflush(stdout);
        if (++buffer.produce_pos >= buffer.capacity)
            buffer.produce_pos = 0;
        if (item > MAX)
            produceStop = 1;
        pthread_mutex_unlock(&mutex);
        pthread_cond_signal(&consumer_cv);
    }
    return NULL;
}

// 消费者线程函数
void *consume() {
    while (1) {
        pthread_mutex_lock(&mutex);
        while (buffer.product_num <= 0 && !consumeStop)
            pthread_cond_wait(&consumer_cv, &mutex);
        if (consumeStop) {
            pthread_mutex_unlock(&mutex);
            break;
        }
        int item = buffer.data[buffer.consume_pos];
        printf("Consume: %d at position %d, buffer contains %d product now.\n", item, buffer.consume_pos, --buffer.product_num);
        fflush(stdout);
        if (++buffer.consume_pos >= buffer.capacity)
            buffer.consume_pos = 0;
        if (item > MAX) 
            consumeStop = 1;
        pthread_mutex_unlock(&mutex);
        pthread_cond_signal(&producer_cv);
    }
    return NULL;
}

int main() {
    buffer.capacity = 20;
    buffer.data = (int *) malloc(20 * sizeof(int));
    buffer.product_num = 0;
    buffer.produce_pos = 0;
    buffer.consume_pos = 0;
    pthread_mutex_init(&mutex, NULL);
    pthread_cond_init(&consumer_cv, NULL);
    pthread_cond_init(&producer_cv, NULL);

    pthread_t producer_threads[10];
    pthread_t consumer_threads[10];
    for (int i = 0; i < 10; i++)
        pthread_create(&producer_threads[i], NULL, produce, NULL);
    for (int i = 0; i < 10; i++)
        pthread_create(&consumer_threads[i], NULL, consume, NULL);

    for (int i = 0; i < 10; i++)
        pthread_join(producer_threads[i], NULL);
    for (int i = 0; i < 10; i++)
        pthread_join(consumer_threads[i], NULL);

    pthread_mutex_destroy(&mutex);
    pthread_cond_destroy(&producer_cv);
    pthread_cond_destroy(&consumer_cv);

    return 0;
}
```

![截屏2022-12-09 17.05.04](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202212091705109.png)
