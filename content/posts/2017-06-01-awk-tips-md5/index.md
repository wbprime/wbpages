+++
title = "Awk Tips: md5"
description = "Awk Tips: md5"
date = 2017-06-01T09:43:06+08:00
draft = false
template = "page.html"
[taxonomies]
categories =  ["Notes"]
tags = ["awk", "tech", "md5"]
+++

# Awk 之 md5

## 用例

有这么一个需求：文本文件中每一行是用户名和密码，现在需要对文件进行变换，输出用户名和对应的密码的md5值。

原始文件("input.txt")如下：

```
user1 user1_password
user2 user2_password
user3 user3_password
user4 user4_password
user5 user5_password
user6 user6_password
user7 user7_password
```

## 解决

可以使用支持GNU扩展特性的awk来完成这个任务。

awk 脚本如下:

```
#!/usr/bin/awk -f

BEGIN {
    md5cmd = "md5sum";
}

{
    printf $2 |& md5cmd;
    close(md5cmd, "to");

    md5cmd |& getline md5result;
	close(md5cmd);

    print $1, substr(md5result, 1, 32);
}
```

脚本另存为 "md5.awk" 。

在终端中输入:

```
> awk -f md5.awk input.txt
```

输出结果如下：

```
user1 b4ca6e90dcef1196a20930c2d9ecfbc0
user2 049914ab3268e59eb90526f64a5322d9
user3 99675fb0c81fdf505a95318e4c72b685
user4 5f3cca1f85aa649323c336546b3c7cc0
user5 41a87fa27533b238ed77824267259837
user6 758b093168f255636ad83f5f076213e6
user7 c1293b0e2400acb1461d25dbe4c6e75c
```

## 释义

解决方案使用了GNU awk扩展的协程功能，详解参见 [Two-Way Communications with Another Process](https://www.gnu.org/software/gawk/manual/html_node/Two_002dway-I_002fO.html#Two_002dway-I_002fO）。

具体来说，`printf $2 |& md5cmd;` 这一行会将用户密码输出，同时 awk 会使用 sh 启动 md5cmd 进程，并将该进程的标准输入与 awk 的标准输出相连接；md5cmd 读取标准输入之后会输出其 md5sum 的结果到标准输出；`md5cmd |& getline md5result;` 这一行会将 md5cmd 进程的标准输出作为输入，读取一行文本并存储到变量 md5result 中。

因为这些语句是对每一行输入文本都会执行的，所以需要在合适的时候关闭 md5cmd 进程的输入输出文件标识符。`close(md5cmd, "to");` 关闭输入到 md5cmd 进程的文件标识符；`close(md5cmd)` 关闭剩下的其他文件标识符。如果不关闭的话，在文本文件行数很多的情况下，可能会出现由于大量打开一次性文件标识符而不释放导致可用文件标识符被耗尽的错误。
