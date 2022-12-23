1. 

```c
   #include <stdio.h>
   #include <pthread.h>
   #include <semaphore.h>
   
   sem_t sem_A, sem_B;
   
   void *print_B(void *arg) {
   	for (int i = 0; i < 50; ++i) {
   		sem_wait(&sem_B);
   		printf("thread2: %d\n", 2 * i + 1);
   		sem_post(&sem_A);
   	}
   }
   
   int main() {
   	sem_init(&sem_A, 0, 1);
   	sem_init(&sem_B, 0, 0);
   	pthread_t tid;
   	pthread_create(&tid, NULL, print_B, NULL);
   	for (int i = 0; i < 50; ++i) {
   	 	sem_wait(&sem_A);
   		printf("thread1: %d\n", 2 * i);
   		sem_post(&sem_B);
   	}
   	pthread_join(tid, NULL);
   }
```

2. 
```c
   #include <stdio.h>
   #include <pthread.h>
   
   #define thread_num 5
   
   
   pthread_mutex_t mu;
   int counter;
   
   void *add(void *arg) {
   	for (int i = 0; i < 1000000; i++) {
   		pthread_mutex_lock(&mu);
   		++counter;
   		pthread_mutex_unlock(&mu);
   	}
   }
   
   
   int main() {
   	pthread_mutex_init(&mu, NULL);
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

3. 

```c

 #include <semaphore.h>

typedef struct {
    // User defined data may be declared here.
    int h;
    sem_t sem_h, sem_o;
} H2O;

H2O* h2oCreate() {
    H2O* obj = (H2O*) malloc(sizeof(H2O));
    // Initialize user defined data here.
    sem_init(&obj->sem_h, 0, 0);
    sem_init(&obj->sem_o, 0, 1);
    obj->h = 0;

    return obj;
}

void hydrogen(H2O* obj) {
    sem_wait(&obj->sem_h);
    // releaseHydrogen() outputs "H". Do not change or remove this line.
    releaseHydrogen();
    if(++(obj->h)==2) {
        obj->h = 0;
        sem_post(&obj->sem_o);
    }
}

void oxygen(H2O* obj) {
    sem_wait(&obj->sem_o);
    // releaseOxygen() outputs "O". Do not change or remove this line.
    releaseOxygen();
    sem_post(&obj->sem_h);
    sem_post(&obj->sem_h);
}

void h2oFree(H2O* obj) {
    // User defined data may be cleaned up here.
    free(obj);
}
```

4. 

```c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>

#define SIZE 20
#define MAX 100
#define PRODUCE_THREAD_NUM 10
#define CONSUME_THREAD_NUM 10
struct  Container{
    int *data;
    int producePos;
    int consumePos;
    int productNumber;
    int size;
} container;

int indexGrow(int size, int i) {
    return ++i >= size ? 0 : 1;
}

void put(int item) {
    container.data[container.producePos] = item;
    container.producePos = indexGrow(container.size, container.producePos);
    container.productNumber++;
    printf("produce: %d\n", item);
    fflush(stdout);
}

int take() {
    int result = container.data[container.consumePos];
    container.consumePos = indexGrow(container.size, container.consumePos);
    container.productNumber--;
    printf("consume: %d\n", result);
    fflush(stdout);
    return result;
}

int canConsume() {
    return container.productNumber >= 1;
}

int canProduce() {
    return container.productNumber < container.size;
}

int produceStop, consumeStop;

int mCount;

pthread_mutex_t mutex;

pthread_cond_t canConsumeCond, canProduceCond;

void init() {
    container.size = SIZE;
    container.data = (int *) malloc(container.size * sizeof(int));
    container.consumePos = container.producePos = container.productNumber = 0;

    pthread_mutex_init(&mutex, 0);
    pthread_cond_init(&canConsumeCond, 0);
    pthread_cond_init(&canProduceCond, 0);
    produceStop = consumeStop = 0;
    mCount = 0;
}

void *produce() {
    while(1) {
        pthread_mutex_lock(&mutex);
        while (!canProduce() && !produceStop) {
            pthread_cond_wait(&canProduceCond, &mutex);
        }
        if (produceStop) {
            pthread_mutex_unlock(&mutex);
            break;
        }
        int item = ++mCount;
        put(item);
        if (item >= MAX) {
            produceStop = 1;
        }
        pthread_cond_signal(&canConsumeCond);
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

void *consume() {
    while(1) {
        pthread_mutex_lock(&mutex);
        while (!canConsume() && !consumeStop) {
            pthread_cond_wait(&canConsumeCond, &mutex);
        }
        if (consumeStop) {
            pthread_mutex_unlock(&mutex);
            break;
        }
        int item = take();
        if (item >= MAX) {
            consumeStop = 1;
        }
        pthread_cond_signal(&canProduceCond);
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

void closeResources() {
    pthread_mutex_destroy(&mutex);
    pthread_cond_destroy(&canConsumeCond);
    pthread_cond_destroy(&canProduceCond);
}

int main() {
    init();
    pthread_t produceThread[PRODUCE_THREAD_NUM];
    pthread_t consumeThread[CONSUME_THREAD_NUM];
    for (int i = 0; i < PRODUCE_THREAD_NUM; i++) {
        pthread_create(produceThread + i, 0, produce, 0);
    }
    for (int i = 0; i < CONSUME_THREAD_NUM; i++) {
        pthread_create(consumeThread + i, 0, consume, 0);
    }
    for (int i = 0; i < PRODUCE_THREAD_NUM; i++) {
        pthread_join(produceThread[i], 0);
    }
    for (int i = 0; i < CONSUME_THREAD_NUM; i++) {
        pthread_join(consumeThread[i], 0);
    }
    closeResources();
    return 0;
}
```

