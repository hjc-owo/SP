# lab01 答题卡

## 1. 使用`apt-get `命令安装`vim`（不需要作答）

## 2. 打开终端（Terminal），以下所有实验请在终端中进行（不需要作答）。

## 3. 查看当前登录的用户和所在的目录，分别列出所使用的命令。

答：

```shell
who
pwd
```

![Screenshot from 2022-09-05 21-29-36](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209052134713.png)

## 4. 创建一个空目录`test`，进入该目录，列出所使用的命令。

答：

```shell
mkdir test
cd test
```

![Screenshot from 2022-09-05 21-30-52](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209052135741.png)

## 5. 在`test`目录构建出以下结构的文件目录，并列出所使用的命令。

答：

```shell
mkdir a
mkdir a/b
touch a/b/c.txt
touch a/b/d.c
mkdir a2
touch a.rs
```

![Screenshot from 2022-09-05 21-42-02](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209052143375.png)

## 6. 将`test`目录变成以下结构，并列出所使用的命令。**能否通过两条命令实现？**Hint: cp + 通配符

答：

```shell
cp -r a/b a/b2
cp -r a/* a2
```

![Screenshot from 2022-09-05 22-00-36](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209052200114.png)

## 7. 使用`wget https://gitee.com/qq994876761/22_Fall_sp_labs/raw/main/lab01/practice.zip`获取`practice.zip`文件，查看解压后的`practice.c`的最后一行，写出`secret`的值。

答：

```
secret=20212121
```

![Screenshot from 2022-09-05 22-03-28](https://raw.githubusercontent.com/hjc-owo/hjc-owo.github.io/img/202209052204209.png)

## 8. 使用vim打开`practice.c`，**依次**完成以下操作，写出每次使用的命令，**不要进入编辑模式**。

#### (1) 删除第13行的代码并保存修改。

答：

```
:13d
:w
```

#### (2) 删除最后一行的`secret`的注释并保存。

答：

```
:$d
:w
```

#### (3) 删除第1000行到第1020行（共21行）并保存。

答：

```
:1000,1020d
:w
```

#### (4) 将第50行复制到第100行并保存。

答：

```
:50 copy 100
:w
```

#### (5) 将全文的`static`替换成`stastic`并保存。

答：

```
:%s/static/stastic/gc
:w
```

#### (6) 退出vim。

答：

```
:q
```

