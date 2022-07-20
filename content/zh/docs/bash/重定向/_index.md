---
title: "重定向和管道"
linkTitle: "重定向和管道"
weight: 1
description:
---

## stdin stdout 和 stderr

stdin stdout 和 stderr 是 Linux 中的三个特殊的文件描述符。

| 文件描述符  | 英文  | 中文  |
|---|---|---|
| 0 | stdin   | 标准输入  | 
| 1 | stdout  | 标准输出  |  
| 2 | stderr  | 标准错误  |   


stdout 和 stderr 缺省输出到 terminal window 上。

## 重定向

重定向操作符 `>` 和 `>>` 可以把 stdout 从 terminal window 重定向到文件中。

* `>` : 重定向 stdout 到文件，会覆盖已有文件的内容。
* `>>` : 重定向 stdout 到文件，追加到已有文件内容上。

注意：缺省只会重定向 stdout，如果也需要重定向 stderr，可以把 stderr 重定向到 stdout，然后再重定向到文件中。

假如有一个 error.sh 脚本，内容如下：

```bash
#!/bin/bash

echo "About to try to access a file that doesn't exist"
cat bad-filename.txt
```

执行下面的命令：

```bash
➜  ~ ./error.sh>log
cat: bad-filename.txt: No such file or directory
➜  ~ cat log
About to try to access a file that doesn't exist
```
可以看到 stdout 被重定向到了 log 文件，而 stderr 还是显示在 terminal 中。该命令是 ```./error.sh 1>log```的简写形式。

如果也要重定向 stderr，可以这样做：

```bash
➜  ~ ./error.sh>log 2>&1
➜  ~ cat log
About to try to access a file that doesn't exist
cat: bad-filename.txt: No such file or directory
```
`2>&1` 表示把 stderr 重定向到 stdout。这里的 2 和 1 分别是 stderr 和 stdout 的文件描述符。为了区别普通文件，当将 stderr 和 stdout 作为重定向目的地时，需要在文件描述符前加上 `&`。

该命令也可以写作 ```./error.sh 1>log 2>&1```。 ```./error.sh 1>>log 2>>log``` 也可以达到类似的目的，但不会覆盖文件内容。

如果要把 stdout 和 stderr 分别重定向到不同的文件，可以这样写：```./error.sh 1>out.log 2>error.log```

`/dev/null` 是一个特殊文件，任何写入该文件的内容都会被丢弃掉。将标准输出重定向到该文件可以用于清空 terminal 的命令行输出。```./error.sh>/dev/null 2>&1```

## 管道

管道 `|` 可以把前一个命令的标准输出（stdout）作为后一个命令的标准输入（stdin）。

例如通过 grep 来查找命令标准输出中的文本。

```bash
~ ./error.sh | grep -o file
file
cat: bad-filename.txt: No such file or director
```

如果要需要同时查找 stdout 和 stderr 中的文本，可以先把 stderr 重定向到 stdout。

```bash
➜  ~ ./error.sh 2>&1 | grep -o file
file
file
file
```
上面的命令也可以写作 ```./error.sh |& grep -o file```

并不是所有的文件都支持从标准输入中读取数据，对于不支持读取标准输入的命令，可以采用下面的命令将标准输出放到一个缓冲区中，然后再用后一个命令打开。

``` ./error.sh | vi - ```