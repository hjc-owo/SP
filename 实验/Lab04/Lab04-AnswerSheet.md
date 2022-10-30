# Lab04 - Assignment

> 姓名：胡峻诚
>
> 学号：19241027

## 1. 文件 I/O 函数

(1)创建一个文件，内容为你的姓名的全拼（如张三同学，文件中的内容即为`zhangsan`)。编写 c 语言程序实现以下功能：首先打开该文件并输出文件的内容，之后将文件的内容修改为`May the force be with you, ${姓名全拼}!`，比如`May the force be with you, zhangsan!`，输出修改后文件的内容，最后关闭文件。要求使用到`open()` `read()` `write()` `close()`函数。请详细叙述你的操作过程以及操作过程的截图，并给出你所编写的 C 程序的代码。

>操作过程说明与截图：
>
>1. 编写 C 程序 `IO1.c`
>2. 填写 `test.txt`，内容为 `Hujuncheng`
>3. 编译 `IO1.c`
>4. 运行 `IO1`， 输出 `May the force be with you, Hujuncheng!`
>
>![Screenshot from 2022-10-19 17-06-36](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210191706482.png)
>
>C 程序代码：
>
>```c
>#include <unistd.h>
>#include <stdio.h>
>#include <fcntl.h>
>#include <string.h>
>#include <stdlib.h>
>
>int main() {
>	char s[50] = "May the force be with you, ";
>	char name[15] = {0};
>	char *s2 = "!";
>	int fd;
>	if ((fd = open("test.txt", O_RDWR)) == -1) {
>		perror("Open error.");
>		exit(1);
>	}
>	ssize_t size = read(fd, name, 15);
>	printf("%s", name);
>	if (size == -1) {
>		perror("Read error.");
>		exit(1);
>	}
>
>	strcat(s, name);
>	strcat(s, s2);
>	lseek(fd, 0, SEEK_SET);
>	write(fd, s, strlen(s));
>	close(fd);
>	return 0;
>}
>```

(2)使用`fopen()` `fread()` `fwrite()` `fclose()`函数再次实现上述要求。请详细叙述你的操作过程以及操作过程的截图，并给出你所编写的 C 程序的代码。

>操作过程说明与截图：
>
>1. 编写 C 程序 `IO2.c`
>2. 填写 `test.txt`，内容为 `Hujuncheng`
>3. 编译 `IO2.c`
>4. 运行 `IO2`， 输出 `May the force be with you, Hujuncheng!`
>
>![Screenshot from 2022-10-19 18-37-38](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210191838464.png)
>
>C 程序代码：
>
>```c
>#include <stdio.h>
>#include <string.h>
>
>int main() {
>	char s[50] = "May the force be with you, ";
>	char name[15] = {0};
>	char *s2 = "!";
>	FILE *file = fopen("test.txt", "r+");
>	fread(name, 15, 1, file);
>	printf("%s", name);
>	strcat(s, name);
>	strcat(s, s2);
>	fseek(file, 0, SEEK_SET);
>	fwrite(s, strlen(s), 1, file);
>	fclose(file);
>	return 0;
>}
>```

## 2. 目录

编写一个 myls 程序，要求输入一个参数代表指定目录，例如./listdir/，打印目录下所有文件的名称。

（Hint：参考课本例 4-8）

> ```c
> #include <sys/types.h>
> #include <dirent.h>
> #include <stdio.h>
> 
> int main(int argc, char **argv) {
> 	DIR *dir_ptr = opendir(argv[1]);
> 	struct dirent *dir;
> 	if (dir_ptr != NULL) {
> 		while ((dir = readdir(dir_ptr)) != NULL)
> 			printf("%s\n", dir->d_name);
> 		closedir(dir_ptr);
> 	}
> 	return 0;
> }
> ```
>
> ![Screenshot from 2022-10-19 18-47-10](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210191847324.png)

## 3.  链接

（1）创建文件`~/srcfile`，使用 `ln` 命令创建 `srcfile` 的软链接文件 `~/softlink` ，给出使用的命令；使用 `ls -l` 查看 `~` ，观察 `softlink` 的文件大小，并解释为什么；使用 `ln` 命令创建 `srcfile`的硬链接文件 `~/hardlink` ，给出使用的命令；使用 `ls -l` 观察 `srcfile` 硬链接数的变化。

>如图所示
>
>![Screenshot from 2022-10-19 18-55-14](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210191901103.png)
>
>可见 `softlink` 的文件大小为 7 字节，因为软连接存储源文件的路径。

（2）查看`srcfile`链接文件的索引节点号和文件内容。接下来修改源文件、硬链接文件、软链接文件，查看其他两个文件内容的变化。然后删除源文件，观察硬链接文件和软链接文件的变化，请给出操作过程的截图以及得出的结论。

>![Screenshot from 2022-10-19 19-42-54](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202210191943045.png)
>
>`srcfile`链接文件的索引节点号为`262277`，文件内容为空。
>
>修改源文件、硬链接文件、软链接文件时，只有源文件和硬链接文件会有改变，软连接文件没有变化。
>
>删除源文件，硬连接数减一，硬链接还可以正常打开，但是软连接已经不能正常打开。



