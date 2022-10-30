# Lab 02 Answer Sheet

> 姓名：胡峻诚
> 学号：19241027

## 1. GCC

完成以下操作，每一个操作限用一条指令，且不能更改文件位置：

- 将 `./main.c` 编译为 `./main.o`（仅编译）

  > 指令：`gcc -c main.c -I include`

- 将 `./v1/dog.c` 编译为 `./v1/dog.o`（仅编译）

  > 指令：`gcc -o ./v1/dog.o -c ./v1/dog.c -I include`

- 将 `./v2/dog.c` 编译为 `./v2/dog.o`（仅编译）

  > 指令：`gcc -o ./v2/dog.o -c ./v2/dog.c -I include`

- 将 `./v1/dog.o` 与 `./main.o` 链接为 `./dog1`

  > 指令：`gcc ./v1/dog.o main.o -o dog1`

- 将 `./v2/dog.o` 与 `./main.o` 链接为 `./dog2`

  > 指令：`gcc ./v2/dog.o main.o -o dog2`

- 执行 `./dog1`

  > 指令：`./dog1`
  >
  > 输出结果：I'm a dog, my name is P-A-I-M-O-N.
  >
  > ![Screenshot from 2022-09-12 19-22-05](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209121922826.png)

- 执行 `./dog2`

  > 指令：`./dog2`
  >
  > 输出结果：I'm a really really beautiful dog, my holy name is P-A-I-M-O-N.
  >
  > ![Screenshot from 2022-09-12 19-23-02](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209121933975.png)

## 2. 静态和动态库的操作

#### 静态库

- 执行下面的命令

  ```shell
  ar cr libdog.a ./v1/dog.o
  gcc -o main main.o -L. -ldog
  ```

  说明上述两个命令完成了什么事？

  > `ar cr libdog.a ./v1/dog.o`表示生成一个libdog.a的静态库，这里参数c表示创建新的静态库文件，r表示将目标文件加入静态库中，如果之前静态库中有与之同名的文件，则将其删除。
  >
  > `gcc -o main main.o -L. -ldog`表示编译main.o时，将该文件夹下的文件添加到库文件搜索路径当中，同时制定了编译时所搜索的库的名称。
  
- 查看`main`文件大小，并记录

  > 16760B
  >
  > ![Screenshot from 2022-09-12 19-31-24](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209121933829.png)

- 执行`./main`

- 执行以下命令：

  ```sh
  ar cr libdog.a ./v2/dog.o
  ./main
  ```

  和上面执行的结果有不同之处吗？

  > 两次结果一样，没有不同之处。
  >
  > ![Screenshot from 2022-09-12 19-32-45](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209121933185.png)

#### 动态库

- 执行下面的命令

  ```shell
  gcc -c -fPIC v1/dog.c -o v1/dog.o -I include
  gcc -c -fPIC v2/dog.c -o v2/dog.o -I include
  gcc -shared -fPIC -o libdog.so v1/dog.o
  gcc main.c libdog.so -o main -I include
  ```

- 查看 `main` 文件大小，并记录

  > 16680B
  >
  > ![Screenshot from 2022-09-12 19-34-08](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209121934216.png)

- 执行`./main` 前，需要将库文件路径设置为当前路径：

  ```sh
  LD_LIBRARY_PATH=.
  export LD_LIBRARY_PATH
  ```

- 执行 `./main`

- 执行以下命令：

  ```sh
  gcc -shared -fPIC -o libdog.so v2/dog.o
  ./main
  ```

  和上面执行的结果有不同之处吗？

  > 两次结果不同。
  >
  > ![Screenshot from 2022-09-12 19-35-24](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209121935244.png)

- 和静态库操作结果进行对比，同时分析 `main` 文件的大小，你能得出什么结论？

  > 1. 静态链接库：以.a为后缀。当要使用时，链接器会找出程序所需要的函数，然后将它们拷贝到可执行文件，由于这种拷贝是完整的，所以一旦链接成功，静态链接库也就不需要了。
  > 2. 动态链接库：以.so为后缀。某个程序在运行中要调用某个动态链接库函数的时候，操作系统首先会查看所有正在运行的程序，看在内存中是否已经有此函数的拷贝了。如果有，则让其共享一个拷贝；否则，链接载入。在程序运行的时候，被调用的动态链接库函数被安置在内存的某个地方，所有调用它的程序将指向这个代码段。
  > 3. 所以综合来看，静态库链接的main文件大小略大于动态库链接的main文件大小。

## 3. GDB

（2）在 main 函数处设置断点（写出命令）

```shell
b main
```

（7）在程序第 10 行加入断点（写出命令）

```shell
b 10
```

（10）使用 print 命令打印 i 和 factorial 的值（写出命令）

```shell
print i
print factorial
```

说明源程序中存在的错误。

> factorial没有初始化。

你更喜欢使用 printf 还是 gdb 调试？为什么？

> 显然是printf调试，一是习惯，二是不用自己输入命令就可以在想要的地方输出。
>
> 但是gdb调试也有其优点：
>
> 1. 可以任意设置断点并从断点处运行
> 2. 可以随意查看自己想看的变量
> 3. 可以将变量设置为自己想要的值进行调试
> 4. 循环体可以一遍一遍的执行，也可以直接一次执行跳过

## 4. make

`./Makefile` 文件内容：

```makefile
main: fun1.o fun2.o main.o
        gcc -o main main.o fun1.o fun2.o -L lib -ldy

fun1.o: src/fun1.c
        gcc -c src/fun1.c -o fun1.o -I include

fun2.o: src/fun2.c
        gcc -c src/fun2.c -o fun2.o -I include

main.o: src/main.c
        gcc -c src/main.c -o main.o -I include

clean: 
        rm -f *.o main

```

`main` 可执行程序输出了什么？

```
hello world
this is fun1
this is fun2
HIDE AND SEEK, FIND YOU NEXT WEEK!
```

![Screenshot from 2022-09-12 21-00-43](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209122100548.png)

