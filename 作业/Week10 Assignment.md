# Week10 Assignment

> 班级：202111
>
> 学号：19241027
>
> 姓名：胡峻诚

*阅读教材第六章并查阅网络资料，回答以下问题。*

#### 1. 编写一个程序，实现这样的功能：搜索2~65535之间所有的素数并保存到数组中，用户输入^C信号时，程序打印出最近找到的素数。

> ```c
> #include <stdio.h>
> #include <signal.h>
> 
> int isPrime(int n) {
> 	for (int i = 2; i < n; i++) 
> 		if (n % i == 0)
> 			return 0;
> 	return 1;
> }
> 
> int prime[65555], now;
> 
> void sigHandler(int signalNum) {
> 	printf("The nearest prime is: %d\n", prime[now]);
> }
> 
> int main() {
> 	signal(SIGINT, sigHandler);
> 	for (int i = 2; i <= 65536; i++) 
> 		if (isPrime(i))
> 			prime[++now] = i;
> 	return 0;
> }
> ```
>
> ![Screenshot from 2022-11-02 14-04-02](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202211021404029.png)

#### 2.简述什么是可靠信号和不可靠信号，并实验SIGINT信号是可靠的还是不可靠的。

> - 可靠信号：多个信号发送到进程时，没来得及处理的信号排入进程的队列，依次处理；
> - 不可靠信号：多个信号产生时，无法来得及处理，造成信号丢失。
>
> 信号值小于`SIGRTMIN`为不可靠信号，在`SIGRTMIN-SIGRTMAX`之间的为可靠信号。使用`kill -l`查看所有信号列表。
>
> ![Screenshot from 2022-11-02 14-11-43](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202211021412193.png)
>
> 由此可知，`SIGINT`是不可靠的。

#### 3. 在执行 ping http://www.people.com.cn 时，假设该网站是可 ping 通的，但是在输入^\时，ping 命令并没有结束而是显示 ping 的成功率，但是输入^C时，ping 程序却被退出，请解释发生这一现象的原因。

> 因为`ping`代码信号处理的实现如下：
>
> `set_signal(SIGINT, sigexit);`
>
> `set_signal(SIGALRM, sigexit);`
>
> `set_signal(SIGQUIT, sigstatus);`
>
> 也就是说`SIGINT(^C)`的处理函数是`sigexit`，让`ping`命令退出；
>
> 而`SIGQUIT(^\)`的处理函数是`sigstatus`，即显示状态，所以会显示`ping`的成功率。

#### 4.简述什么是不可重入函数，为什么信号处理函数中要尽量避免包含不可重入函数？

> 某个函数可被多个任务并发使用，而不会造成数据错误，则该函数具有可重入性；否则，是不可重入函数。
>
> 信号处理函数中，避免使用不可重入函数，因为信号处理函数有可能被调用多次。若处理函数使用了不可重入函数而变成不可重入时，则必须阻塞信号，若阻塞信号，则信号有可能丢失。

#### 5.编写一个程序，实现这样的功能：程序每隔1秒就给自身发送一个信号，程序接收到该信号后，打印出当前的时间。

提示: 

- 发送的信号可以是任何能实现功能的信号。
- 打印时间的格式不做限制，任何形式都是正确的。

> ```c
> #include <stdio.h>
> #include <signal.h>
> #include <unistd.h>
> #include <string.h>
> #include <time.h>
> 
> void sigHandler(int sig) {
> 	time_t t;
> 	time(&t); 
> 	puts(ctime(&t));
> }
> 
> int main() {
> 	while (1) {
> 		signal(SIGINT, sigHandler);
> 		sleep(1);
> 		raise(SIGINT);
> 	}
> }
> ```
>
> ![Screenshot from 2022-11-02 17-04-20](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202211021704421.png)

#### 6.编写程序实现如下功能：程序 A.c 按照用户输入的参数定时向程序 B.c 发送信号，B.c 程序接收到该信号后，打印输出一条消息。

运行过程如下：

```sh
./B value& # 此时，输出进程 B 的 PID 号，value 表示要输出的参数。
./A processBPID timerVal # 第一个参数表示进程 B 的 PID，第二个参数为定时时间。
```

> `A.c`
>
> ```c
> #include <signal.h>
> #include <stdio.h>
> #include <unistd.h>
> 
> int main(int argc, char *argv[]) {
> 	int pid;
> 	sscanf(argv[1], "%d", &pid);
> 	int time;
> 	sscanf(argv[2], "%d", &time);
> 	while (1) {
> 		kill(pid, 2);
> 		sleep(time);
> 	}
> 	return 0;
> }
> ```
>
> `B.c`
>
> ```c
> #include <signal.h>
> #include <stdio.h>
> #include <unistd.h>
> #include <string.h>
> 
> char buf[186];
> 
> void sigHandler(int sig, siginfo_t *info, void *mask) {
> 	puts(buf);
> }
> 
> int main(int argc, char *argv[]) {
> 	strcpy(buf, argv[1]);
> 	struct sigaction act;
> 	sigset_t new, old;
> 	sigemptyset(&new);
> 	sigaddset(&new, SIGINT);
> 	sigprocmask(SIG_BLOCK, &new, &old);
> 	act.sa_sigaction = sigHandler;
> 	act.sa_flags = SA_SIGINFO;
> 	sigaction(SIGINT, &act, NULL);
> 	sigaction(SIGRTMIN, &act, NULL);
> 	sigprocmask(SIG_SETMASK, &old, NULL);
> 	while (1);
> }
> ```
>
> ![Screenshot from 2022-11-02 17-52-21](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202211021753937.png)

