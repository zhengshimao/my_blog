---
title: 封个bash小脚本 | 由文件（夹）的相对路径获取绝对路径
tags:
  - bash脚本
  - linux
keywords: “bash脚本,linux”
categories:
 - 归档

date: 2022-10-22
  
toc: true
draft: false
url: post/get_pwd.html
---

# 封个bash小脚本 | 由文件（夹）的相对路径获取绝对路径

## 介绍

在写命令行的时候，绝对路径在服务器上调用更加方便（也看个人习惯）。

我搜了下，没有找到直接可用的工具。

感觉很简单，自己昨天写一个bash脚本。这个第一版本的脚本只能接受参数传递，如果要使用管道符，必须借用`xargs` 给脚本传递参数。但是经常用的话还要加`xargs` ，未免繁琐了点。

本次修改要求：

- 使其接受多个参数：遍历变量`$*` ,`$@` 或`"$@"` 。

- 使其直接接受管道符传递参数：关键是嵌入`while read line` 。

## 使用

由于经常使用，我将脚本直接放在了已经加入环境变量的路径下`~/scripts` 。并将其命名为`rd` （= real  directory）

### 获取单个文件绝对路径

```sh
$ rd 1.qc.sh
/home/data/vip13t28/workspace/pig/1.qc.sh

```

### 获取单个文件夹绝对路径

```sh
$ rd 2.clean_reads/
/home/data/vip13t28/workspace/pig/1.qc.sh
/home/data/vip13t28/workspace/pig/2.clean_reads
```

### 获取多个文件（夹）

```sh
$ rd 4.rseqc/ 5.stringtie_gtf  nohup.out 
/home/data/vip13t28/workspace/pig/4.rseqc
/home/data/vip13t28/workspace/pig/5.stringtie_gtf
/home/data/vip13t28/workspace/pig/nohup.out

```

### 获取多个文件绝对路径

```sh
$ ls data/SRR18059* |rd
/home/data/vip13t28/workspace/pig/data/SRR1805929_1.fastq.gz
/home/data/vip13t28/workspace/pig/data/SRR1805929_2.fastq.gz
/home/data/vip13t28/workspace/pig/data/SRR1805930_1.fastq.gz
/home/data/vip13t28/workspace/pig/data/SRR1805930_2.fastq.gz
/home/data/vip13t28/workspace/pig/data/SRR1805931_1.fastq.gz
/home/data/vip13t28/workspace/pig/data/SRR1805931_2.fastq.gz
/home/data/vip13t28/workspace/pig/data/SRR1805932_1.fastq.gz
/home/data/vip13t28/workspace/pig/data/SRR1805932_2.fastq.gz
/home/data/vip13t28/workspace/pig/data/SRR1805933_1.fastq.gz
/home/data/vip13t28/workspace/pig/data/SRR1805933_2.fastq.gz

```

## 代码

```sh
#!/usr/bin/env bash

# 定义函数，从相对路径获取绝对路径。
function get_pwd(){
    here=$PWD;
    if test -d $1 ;then
    cd $1;
    dir=$PWD;
    echo $dir;
    cd $here ; #同一个子shell下接受多个参数必须记得返回原文件夹，否则会找不到其它参数的文件（夹）。
elif test -f $1; then
    here=$PWD;
    tmp_file=`basename $1`;
    tmp_dir=`dirname $1`;
    cd $tmp_dir;
    dir=$PWD;
    echo "${dir}/${tmp_file}"
    cd $here ; #同一个子shell下接受多个参数必须记得返回原文件夹，否则会找不到其它参数的文件（夹）。
else 
    echo "$1 : No such file or dirctory!"
fi
}

# 
if [[ $# != 0 ]];then
    for i in $@ ;do
        get_pwd $i
    done
else
    while read line;
    do
        get_pwd $line
    done
fi

```

## 小结

- 在遍历参数的时候`$*` , `$@` ,` "$@"` 

- 子shell中路径改变之后记得返回原路径，否则接受多个相对路径的时候只有第一个文件（夹）有结果返回。

- 从**管道符接受参数**的写法：`while read line` 写法。如果参数数量`$#` 为0，则尝试从标准输出用管道符

- 思路真的很重要，基础真的很重要。这点思路尝试写了一个多小时才转过脑筋来。
