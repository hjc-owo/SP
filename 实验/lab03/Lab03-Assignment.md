# Lab03 - Assignment

> 姓名：胡峻诚
>
> 学号：19241027

## 1. 基础

### 1.1 权限

对一个文本文件`file.txt`执行命令：`sudo chmod 777 file.txt`。请解释该命令的含义并写出执行该命令后该文件的权限信息。

> 将`chmod 777 file.txt`这条命令以最高权限运行
>
> `chmod 777 file.txt`表示将`file.txt`的权限信息修改为任何人（所有者、所有组、其他人）都可读可写可执行。
>
> 该文件的权限信息为`rwxrwxrwx`。
>
> ![Screenshot from 2022-10-12 19-40-38](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210122121654.png)

### 1.2 管道、重定向

现有文件`a.txt`，文件内容为

```
Welcome to linux!
```

和文件`f1`，文件内容为

```
Hello World!
```

解释以下命令`cat a.txt | cat >>f1 2>&1`的原理，同时给出执行完命令后f1文件中的内容

> `cat a.txt`表示查看`a.txt`的内容
>
> `|`表示管道，将前面的输出作为后面的输入
>
> `>>f1`表示重定向输出到`f1`原有内容的后面
>
> `2>&1`表示将标准错误从重定向到标准输出指向的文件，在这里也就是`f1`
>
> 执行完命令之后，`f1`文件的内容为：
>
> ```
> Hello World!
> Welcome to linux!
> ```
>
> ![Screenshot from 2022-10-12 19-45-28](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210122122797.png)

### 1.3 环境变量

自己添加一个环境变量，名称是`STUDENT_ID`，值为你的学号，并编写一个C程序来获取该环境变量，并打印出来。**请详细叙述你的操作过程以及操作过程的截图，并给出C程序的代码。**

>先设置环境变量
>
>```shell
>export STUDENT_ID=19241027
>```
>
>然后编写如下所示的C程序
>
>```c
>#include <stdio.h>
>#include <stdlib.h>
>#include <string.h>
>extern char** environ;
>
>int main() {
>	char **p = environ;
>	const char *envName = "STUDENT_ID";
>	for (; *p != NULL; p++) {
>		if (strstr(*p, envName)) {
>			printf("%s\n", getenv(envName));
>		}
>	}
>	return 0;
>}
>```
>
>对该C程序编译运行
>
>```shell
>gcc -o getenv getenv.c
>```
>
>运行截图：
>
>![Screenshot from 2022-10-12 20-14-34](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210122122164.png)

## 2. Shell 编程

### 2.1 脚本解释器

假如在脚本的第一行放入`#!/bin/rm`或者在普通文本文件中第一行放置`#!/bin/more`，然后将文件设为可执行权限执行，看看会发生什么，并解释为什么。

> 脚本的第一行放入`#!/bin/rm`运行后，该脚本文件被删除
>
> 普通文本文件中第一行放置`#!/bin/more`执行后，在标准输出上输出`#!/bin/more`
>
> ![Screenshot from 2022-10-12 20-20-33](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210122125142.png)
>
> 第一行的`#!/bin/*`当中的`*`部分就指明了使用的解释器，运行这个文件的时候就会根据这个解释器去执行
>
> 脚本的第一行放入`#!/bin/rm`，直接运行该文件，相当于执行`rm`命令
>
> 同样的，普通文本文件中第一行放置`#!/bin/more`，直接运行该文件，相当于执行`more`命令

### 2.2 任何位置都可以运行的 Bash

编写一个 bash 脚本，执行该脚本文件将得到两行输出，第一行是你的学号，第二行是当前的日期（考虑使用`date`命令）。

要求：

- 用户可以**在任意位置只需要输入文件名**就可以执行该脚本文件
- **不破坏除用户家目录之外的任何目录结构**，即不要在家目录之外的任何地方增删改任何文件

你的 bash 脚本文件内容：

```bash
#!/bin/bash
echo 19241027
echo `date`
```

除编写该文件之外你进行了哪些操作？

> 在用户根目录下创建`.bash_profile`，内容为
>
> ```bash
> export PATH="~/Lab/lab03:$PATH"
> ```
>
> 即在PATH中添加脚本文件所在目录路径，然后执行 `source ~/.bash_profile`，之后便可以在任何位置执行 `date-19241027.sh`。
>
> ![Screenshot from 2022-10-12 21-15-58](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210122125369.png)

### 2.3 Bash 实战 (1)

完成 [LeetCode: 193. Valid Phone Numbers](https://leetcode.com/problems/valid-phone-numbers/)，并提供你的 AC 代码与截图。

AC 代码：

```bash
grep -P '^(\(\d{3}\) |\d{3}-)\d{3}-\d{4}$' file.txt
```

AC 截图：

> ![截屏2022-10-12 17.01.27](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210121701900.png)

### 2.4 Bash 实战 (2)

完成 [LeetCode: 195. Tenth Line](https://leetcode.com/problems/tenth-line/)，并提供你的 AC 代码与截图。

AC 代码：

```bash
sed -n '10p' file.txt
```

AC 截图：

> ![截屏2022-10-12 17.05.15](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210121705879.png)

### 2.5 Bash 实战 (3) 

编写一个文件 `manage.sh`，其用法如下：

```bash
manage.sh OPERATION
```

其中：

- 参数 OPERATION 有以下选项：
  - `mine`：输出当前路径下（不包括子目录）所有者为当前用户的所有文件名
  - `largest`：输出当前路径下（不包括子目录）最大文件的文件名
  - `expand`**（选做）**：递归地将当前目录中所有文件（无论在哪个子文件夹中）全部移动至当前目录下，并删除所有子文件夹。

对于 `expand` 操作，举个例子：

假设 `/root/lab03` 结构如下：

```bash
/root/lab03/
	|------ a.txt
	|------ b.txt
	|------ c/
		|------ d.txt
		|------ e.txt
	|------ f/
		|------ g/
			|------ h.txt
```

运行 `./manage.sh expand` 后，其结构将变为：

```
/root/lab03/
	|------ a.txt
	|------ b.txt
	|------ d.txt
	|------ e.txt
	|------ h.txt
```

你的代码：

```bash
#!/bin/bash
case $1 in
"mine")
	find ./ -maxdepth 1 -type f -user `whoami`;;
"largest")
	find ./ -maxdepth 1 -type f | sort -n | tail -1;;
"expand")
	mv -n `find ./ -type f` ./ 
	rm -rf `find ./ -maxdepth 1 -type d`;;
esac
```
