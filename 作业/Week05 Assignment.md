# Week05 Assignment

> 姓名：胡峻诚
> 
>学号：19241027

阅读教材第三章内容并查阅网络资料，回答下列问题。

## 1. 权限

在使用`ls -l`命令后，会列出目录的很多信息，例如下述例子

```
[me@linuxbox ~]$ ls -l
total 56
drwxrwxr-x 2  me  me  4096  2007-10-26  17:20  Desktop
drwxrwxr-x 2  me  me  4096  2007-10-26  17:20  Documents
drwxrwxr-x 2  me  me  4096  2007-10-26  17:20  Music
drwxrwxr-x 2  me  me  4096  2007-10-26  17:20  Pictures
drwxrwxr-x 2  me  me  4096  2007-10-26  17:20  Public
drwxrwxr-x 2  me  me  4096  2007-10-26  17:20  Templates
```

请依次完成以下问题

在任一有文件的目录下使用`ls -l`命令并截图

> ![Screenshot from 2022-09-27 16-28-07](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209271628685.png)

在展示的内容中，第一个字段`drwxrwxr-x` 中每一位的含义

> d:表示文件类型；
>
> rwx：表示文件所有者的对该文件所拥有的权限；
>
> rwx：表示文件所属组对该文件所拥有的权限；
>
> r-x：表示其他用户对该文件所拥有的权限。

当我们以普通用户身份进行一些操作时会遇到`Permission denied`的情况，为了解决这个问题，我们有两种办法，分别是切换到`root`用户，或者仅让单条指令以最高权限运行，这两个操作分别需要哪两个命令

> 切换到`root`用户：`su`
>
> 仅让单条指令以最高权限运行：命令前加`sudo`

当我们需要更改文件的权限(读取、写入和执行)时需要用到哪个命令

> `chmod`命令
>
> `chmod [u/g/o/a] [+/-/=] [权限] FileName/DirecName` 

## 2. 环境变量

要实现`echo ${name}`的输出为`super 2021/2121 你的学号`，需要事先用什么命令定义环境变量

> `export name=super\ 2021/2121\ 19241027`
>
> ![Screenshot from 2022-09-27 17-02-49](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209271703783.png)

## 3. 重定向

现有 C 文件`test.c`内容如下

```c
#include <stdio.h>

int main(int argc,char *argv[])
{
    printf("Number of args: %d , Args are:\n",argc);
    int i;
    for(i=0;i<argc;i++)
        printf("args[%d] %s\n",i,argv[i]);
    fprintf(stderr,"This is the message sent to stderr\n");
}
```

要将其转换成为可执行文件`test`需要执行什么命令

> `gcc -o test test.c`

运行`test`文件，并要实现重定向输入`test.in`，重定向输出`test.out`，重定向错误`error.log`的命令为

> `./test.c < test.in > test.out 2> error.log`

先后执行`./test abc 235 2>a.txt` 和 `./test abc 234 >>a.txt`，屏幕上和`a.txt`的内容为什么？

屏幕上的内容为

```
huaye@huaye-virtual-machine:~/Homework/week05$ ./test abc 235 2>a.txt
Number of args: 3 , Args are:
args[0] ./test
args[1] abc
args[2] 235
huaye@huaye-virtual-machine:~/Homework/week05$ ./test abc 234 >>a.txt
This is the message sent to stderr
```

![Screenshot from 2022-09-27 17-14-35](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209271714216.png)

`a.txt` 中的全部内容为

```
This is the message sent to stderr
Number of args: 3 , Args are:
args[0] ./test
args[1] abc
args[2] 234
```

## 4. 管道

请写出命令 `who | wc -l` 的结果并写出管道的用途，或者分析管道的执行过程。

> 结果：`1`
>
> 管道是输入输出重定向的特例，将前一个命令的输出直接连接到后一个命令的输入。这里就是把`who`的输出直接作为`wc -l`的输入。

## 5. Shell 编程

### 1. 简述 shell 中的特殊字符

> 1. `#`：注释。
> 2. `;`：命令分隔符，可以在一行当中写多个命令。
> 3. `,`：连接一系列的算术操作，虽然所有的命令都被运行，但是只有最后一项被返回。
> 4. `\`：转义字符。
> 5. `|`：管道，可以将多个简单的命令连接起来，使一个命令的输出作为另外一个命令的输入，由此来实现更加复杂的功能。
> 6. `> <`：输入输出重定向，用来改变 Shell 获取信息和输出信息的方向。

### 2. Shell 变量有哪几类，各是什么情况下使用？

> - **环境变量**
>
> 定义和系统工作环境有关的变量，用户也可以重新定义该变量。常用于添加或修改环境变量。
>
> - **用户定义变量**
>
> 用户定义的变量，可用于代替表达
>
> - **内部变量（预定义变量）**
>
> 内部变量只能使用而不能修改或重定义，比如 `$#` 为传递给脚本参数的数量，`$$` 为当前进程的进程号，常用于作为暂存文件的名称，以保证不会重复。
>
> | 内部变量 | 说明                                                         |
> | :------: | ------------------------------------------------------------ |
> |   `#`    | Shell 程序位置参数的个数                                     |
> |   `*`    | 以 `IFS` 为分隔，脚本的所有位置参数内容                      |
> |   `?`    | 上一条前台命令执行后返回的状态值，命令执行成功与失败返回值不同 |
> |   `$`    | 当前 Shell 进程的进程 ID 号                                  |
> |   `!`    | 最后一个后台运行命令的进程号                                 |
> |   `0`    | 当前执行的 Shell 程序的名称                                  |
> |   `@`    | 脚本的位置参数内容，从 1 开始，`$@` 被扩展为 `$1`、`$2` 等，分别表示第一个、第二个等的位置参数内容 |
> |   `_`    | Shell 启动时，为正在执行的 Shell 程序的绝对路径；启动后，为上一条命令的最后一个参数 |
>
> 除此之外，常见的变量语法有参数置换变量，能根据不同条件给变量赋不同的值。常与位置参数变量结合使用。
>
> - `变量=${parameter:-word}`：如果 `parameter` 未定义或为 `null`，则用 `word` 置换变量的值，否则用 `parameter` 置换变量的值。
> - `变量=${parameter:=word}`：如果 `parameter` 未定义或为 `null`，则用 `word` 置换 `parameter` 的值然后置换变量的值，否则用 `parameter` 替换变量的值。
> - `变量=${parameter:?word}`：如果 `parameter` 未定义或为 `null`，`word` 被写至标准出错（默认情况下在屏幕显示 `word` 信息）然后退出，否则用 `parameter` 置换变量的值。
> - `变量=${parameter:+word}`：如果 `parameter` 未定义或为 `null`，则不进行替换，即变量值也为 `null`，否则用 `word` 置换变量的值。

### 3. C 语言中的分支语句有 `if-else` 和 `switch-case`，Shell 中是否有相似的语句与其对应？如何使用？

> 1. `if-else`语句
>
>    Shell 中的 if 条件语句分为：单分支 if 语句、双分支 if 语句进而多分支 if 语句，其结构大体与其他程序设计语言的条件语句相同。
>
>    ```shell
>    if [表达式1];then
>    	命令列表1;
>    elif [表达式2];then
>    	命令列表2;
>    else
>    	命令列表3;
>    fi
>    ```
>
> 2. `switch-case`语句
>
>    case 语句可以将一个变量的内容与多个选项进行匹配，若匹配成功，则执行该条件下对应的语句。
>
>    ```shell
>    case 变量 in
>    表达式1)
>    	命令列表1
>    	;;
>    表达式2)
>    	命令列表2
>    	;;
>    ......
>    *)
>    	默认命令列表
>    	;;
>    esac
>    ```

### 4. C 语言中的循环语句有哪些？Shell 中是否有相似的语句与其对应？

> C 语言中的循环语句有`do{}while()`语句`while(){}`语句和`for(){}`语句。
>
> Shell 中
>
> 1. for 循环
>
>    对一个集合中的对象进行循环，并进行相应操作。
>
>    ```shell
>    for name in [ 参数列表 ]
>    do
>    	命令列表
>    done
>    ```
>
>    与 C 语言中的 for 语句也很类似。
>
> 2. while 循环
>
>    while 循环在表达式的值为假时停止循环，否则循环一直进行。
>
>    ```shell
>    while 表达式
>    do
>    	命令列表
>    done
>    ```
>
>    与 C 语言中的 while 语句也很类似。
>
> 3. until 循环
>
>    与 while 循环的格式基本相同，不过 until 循环是在表达式值为真时停止循环，否则循环一直进行。

### 5. 写一个计算 n! 的 Shell 脚本 `frac.sh`

示例输入输出：

```shell
[root@localhost ~]$ ./frac.sh 0
1
[root@localhost ~]$ ./frac.sh 4
24
```

你的脚本内容：

```shell
#!/bin/bash
if [ $1 -eq 0 ];then
	echo 1;
else
	i=1
	number=1
	while [ $i -le $1 ]
	do
		number=$[$number*$i]
		i=$[$i+1]
	done
	echo $number;
fi
```

![Screenshot from 2022-09-27 17-56-13](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209271756023.png)
