---
title: 生信上游流程必会之任务并行ParaFly
tags:
  - 软件使用
  - linux
keywords: “软件使用,linux”
categories:
 - 软件使用

date: 2022-12-16

toc: true
draft: false
url: ability/parafly.html
---

# 生信上游流程必会之任务并行ParaFly

## 介绍

首先我们必须要知道的一个事实是：软件并不是线程设置的越多处理速度越快。这个你可以自行去测试下。

一般我的单个任务设置是`8-12`线程，但是我服务器可用的线程是`45`个。

一般的`for`循环，是可以批量运行，但也是单任务运行。

如何充分利用可用的线程呢？那就是让多个任务一起并行。

你可以单任务设置10个线程，然后一次性运行4个任务。但是手动运行，需要人值守，太牵扯精力。我之前写过利用`for`循环内变量自增控制一次并行任务的数量，但是这样有个缺陷，就是这批的`4`个任务运行完之后才能运行下一批的4个任务。而且每次写这么多代码，有点费脑子。

其实，我们可以不用这么费脑子，因为已经有专业人士写好了任务并行程序。我知道的`linux`下任务并行软件有`ParaFly`和`parallel` 。`ParaFly`的使用对于初学者来说是相对友好的(所以到现在我都没学`parallel`)，本文就介绍下使用`ParaFly`的方法和我的一些经验。

## 安装

github：[ParaFly/ParaFly](https://github.com/ParaFly/ParaFly)  地址：  https://github.com/ParaFly/ParaFly

```sh
conda install -c bioconda parafly 
```

## 使用

```sh
ParaFly -c run_qc.sh -CPU 5 -failed_cmds failed_qc.log
```

- `-c` 指定含有多任务的bash风格的脚本。稍后说下如何准备这个脚本，以及脚本内的内容。
- `-CPU` 帮助文档给出的解释是`number_of_threads` 。这里有一定的迷惑性，实际上应该指定的是一次性并行的任务数量。假如你shell脚本内，每个命令设置了8个线程，你可调用的最大线程是45，那么`-CPU` 选项最大设置是5。也就是一次性并行5个任务，40个左右线程。
- 另外，假如你设置了5个任务并行，其中有1个任务先结束了，该软件会自动调用第6个任务补上，依次类推，会让其始终保持并行5个任务，而不是这5个任务都运行结束之后，再调用5个任务。
- `-failed_cmds` 指定运行失败的命令存放的文件，如果运行全部成功，该文件不会产生。
- 对于运行成功的命令，软件会自动自动写入后缀为`.completed` 文件。
- 如果有部分命令运行失败，再次运行上方命令，则会跳过已经运行成功的命令。

- 还有个有意思的发现，就是这个软件的帮助命令`ParaFly -h` 被视为了标准错误，必须使用` ParaFly --help 2> ParaFly.help`  才能将帮助文档重定向。
- 还有一点经验，就是如果你能利用的最大线程是45，不要一批并行总线程直接搞到45个，因为有些软件对线程控制不够好。还是空出一些线程比较好。

## 准备并行的脚本

`-c` 选项指定的是一个含有多任务的文档，这个文档中一行一个完整命令。

以数据质量检测用的`fastqc` 为例，应该是如下风格的。`-t 6` 指定的就是线程。

```sh
fastqc  -o 1.qc  -t 6 data/SRR1805929_1.fastq.gz data/SRR1805929_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805930_1.fastq.gz data/SRR1805930_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805931_1.fastq.gz data/SRR1805931_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805932_1.fastq.gz data/SRR1805932_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805933_1.fastq.gz data/SRR1805933_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805934_1.fastq.gz data/SRR1805934_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805935_1.fastq.gz data/SRR1805935_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805936_1.fastq.gz data/SRR1805936_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805937_1.fastq.gz data/SRR1805937_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805938_1.fastq.gz data/SRR1805938_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805939_1.fastq.gz data/SRR1805939_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805940_1.fastq.gz data/SRR1805940_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805941_1.fastq.gz data/SRR1805941_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805942_1.fastq.gz data/SRR1805942_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805943_1.fastq.gz data/SRR1805943_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805944_1.fastq.gz data/SRR1805944_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805945_1.fastq.gz data/SRR1805945_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805946_1.fastq.gz data/SRR1805946_2.fastq.gz
fastqc  -o 1.qc  -t 6 data/SRR1805947_1.fastq.gz data/SRR1805947_2.fastq.gz

```

那么这种风格的文档如何准备呢？我能想到的是2种方法：

一种方法是可以使用`for` 循环搭配`echo` 重定向获取；另一种是准备样本文档(最少包含所有样本名称) 让`awk` 搭配其中的`print` 去获取。

下面是我实际运行过程中用`for` 搭配`echo` 准备文档的命令。

```sh
# 获取所有样本名，类似SRR1805947
id=`ls data/*_1.fastq.gz |sed -e 's/.*\///g' -e 's/_.*//g'` 

# for循环生成文档
for i in $id ;do
echo "fastqc  -o 1.qc  -t 6 data/${i}_1.fastq.gz data/${i}_2.fastq.gz" >> run_qc.sh
done

# 或者是
for i in $id ;do
echo "fastqc  -o 1.qc  -t 6 data/${i}_1.fastq.gz data/${i}_2.fastq.gz" 
done > run_qc.sh

# 后台并行任务 nohup command & 
nohup ParaFly -c run_qc.sh -CPU 5 -failed_cmds failed_qc.log & # 不用获取日志，没有。只有成功与失败命令的文档。
```

## 小结

必知必会！仔细体会其特点，尝试去理解两种准备`run_qc.sh ` 文档的区别。

## 帮助文档

```sh
$ ParaFly -h

##########################################################
#
# Usage: ParaFly (opts)
#
# Required: 
#   -c <str>              :filename containing list of bash-style commands to execute.
#   -CPU <int>            :number_of_threads
#
# Optional:
#   -shuffle              :randomly shuffles the command order. 
#   -failed_cmds <str>    :filename to capture failed commands.  default("FailedCommands")
#   -v                    :simple progress monitoring.
#   -vv                   :increased verbosity in progress monitoring.
#
##########################################################

Note: This process creates a file named based on your commands filename with a .completed extension.
This enables a resume functionality, where if rerun, only those commands not completed successfully will be reprocessed.
```

