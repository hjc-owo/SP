# Lab05 - Assignment

> 姓名：胡峻诚
>
> 学号：19241027

## 1. 进程

### 1.1 基础

请写这样一个程序（不是函数）：传入三个参数，传入该程序的第一个参数用以判断该程序进行有理数算术加运算还是减运算（0 表示将要进行加运算，1 表示将要进行减运算），第二个第三个参数分别是加（减）运算的第一第二个元素。（提示：`main(int argc, char* argv[])`，可获取命令行参数）。

```c
#include <stdio.h>
#include <string.h>

int stoi(char s[]) {
    int len = strlen(s);
    int num = 0;
    for (int i = 0; i < len; i++) {
        num *= 10;
        num += s[i] - '0';
    }
    return num;
}

int main(int argc, char *argv[]) {
    if (!strcmp("0", argv[1])) {
        printf("%d\n", stoi(argv[2]) + stoi(argv[3]));
    } else {
        printf("%d\n", stoi(argv[2]) - stoi(argv[3]));
    }
}
```

### 1.2 僵尸进程

僵尸进程有什么危害？编写一个会产生僵尸进程的程序并运行，在终端查看当前进程。然后利用终端杀死该进程。

> 僵尸进程的危害：僵尸进程没有任何代码、数据、堆栈，并不会占用多少资源，但它存在于系统的任务列表。

```c
#include <unistd.h>
#include <stdio.h>

int main() {
    pid_t pid = fork();
    if (pid > 0) {
        sleep(1000);
    }
    return 0;
}
```

> 运行、杀死僵尸进程的过程截图
>
> ![Screenshot from 2022-10-30 19-44-35](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210301945252.png)

## 2. 进程控制

### 2.1 fork 与 wait

编写一段C程序，由父进程创建两个子进程，父进程打印字符`B`，两个子进程分别打印`A`和`C`，并且要求最终的输出为`ABC`。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();
    int status;
    if (pid == 0) {
        putchar('A');
    } else {
        wait(&status);
        pid = fork();
        if (pid == 0) {
            putchar('C');
        } else {
            putchar('B');
        }
    }
    putchar('\n');
    return 0;
}
```

![Screenshot from 2022-10-30 20-14-30](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210302037530.png)

### 2.2 exec族函数

请编写一段C程序，该程序调用调用exec族函数，对当前目录使用`ls -l`。

```c
#include <stdio.h>
#include <unistd.h>

char *s[2] = {"ls", "-l"};

int main() {
    execv("/bin/ls", s);
    return 0;
}
```

![Screenshot from 2022-10-30 20-01-43](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210302002186.png)

### 2.3 综合应用

请你编写一段C程序，该程序按顺序依次调用`题目1.1`中的可执行文件分别执行`1+1`与`1-1`，随后调用`/bin/rm`删除`题目1.1`的可执行文件。

```c
#include <stdio.h>
#include <unistd.h>

char *add[4] = {"./a", "0", "1", "1"};
char *sub[4] = {"./a", "1", "1", "1"};
char *rm[2] = {"rm", "a"};

int main() {
    pid_t pid = fork();
    if (pid == 0) {
        execv("/bin/rm", rm);
    } else {
        pid = fork();
        if (pid == 0) {
            execv("./a", sub);
        } else {
            execv("./a", add);
        }
    }
    return 0;
}
```

![Screenshot from 2022-10-30 20-36-11](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210302036145.png)

## 3. 进程通信

### 3.1 重定向

请编写一个C程序，使用文件描述符与重定向把你的学号输出到`student.txt`文件中

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    close(1);
    open("student.txt", O_RDWR);
    printf("19241027\n");
    return 0;
}
```

![Screenshot from 2022-10-30 20-48-36](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210302049915.png)

### 3.2 综合应用

请使用管道编写素数筛选的并发版本C程序`primes.c`，程序原理见下面的介绍或者查看这个[网站](https://swtch.com/~rsc/thread/)。该程序读入命令行的第一个参数n，随后输出1-n之间的素数。

运行命令

```bash
gcc primes.c -o primes
./primes 7
```

输出为

```
2
3
5
7
```

> 考虑所有小于1000的素数的生成。Eratosthenes的筛选法可以通过执行以下伪代码来模拟：
>
> ```
> p = get a number from left neighbor
> print p
> loop:
>     n = get a number from left neighbor
>     if (p does not divide n)
>         send n to right neighbor
> p = 从左邻居中获取一个数
> print p
> loop:
>     n = 从左邻居中获取一个数
>     if (n不能被p整除)
>         将n发送给右邻居
> ```
>
> ![img](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210302254490.png)
>
> 生成进程可以将数字2、3、4、…、1000输入管道的左端：行中的第一个进程消除2的倍数，第二个进程消除3的倍数，第三个进程消除5的倍数，依此类推。

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void init() {
    for (int i = 2; i <= 1000; i++)
        write(1, &i, sizeof(i));
}

void filter(int p) {
    int n;
    while (read(0, &n, sizeof(n)))
        if (n % p)
            write(1, &n, sizeof(n));
}

void redirect(int k, int pd[]) {
    close(k);
    dup(pd[k]);
    close(pd[0]);
    close(pd[1]);
}

void sink() {
    int p, pd[2];
    if(read(0, &p, sizeof(p))) {
        printf("%d\n", p);
        pipe(pd);
        if (fork()) {
            redirect(0, pd);
            sink();
        } else {
            redirect(1, pd);
            filter(p);
        }
    }
}

int main() {
    int pd[2];
    pipe(pd);
    if (fork()) {
        redirect (0, pd);
        sink();
    } else {
        redirect (1, pd);
        init();
    }
    return 0;
}
```

![Screenshot from 2022-10-30 22-51-35](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210302252430.png)
