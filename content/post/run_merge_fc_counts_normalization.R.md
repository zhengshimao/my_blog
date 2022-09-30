---
title: 封个R脚本 | featurecounts 结果合并、计算FPKM/TPM一步到位
tags:
  - R
  - 转录组
  - 脚本
keywords: “转录组，定量，R脚本”
categories:
 - 归档

date: 2022-09-30
  
toc: true
draft: false
url: run_merge_fc_counts_normalization.R.html
---

## 介绍

linux版本featurecounts定量结果是每个样品一个文件，这里我写了个脚本，合并定量的count矩阵，并根据featurecounts提供的"有效基因长度"计算FPKM和TPM。

我使用的脚本名称：`run_merge_fc_counts_normalization.R`

这里没有测试数据，但是过段时间【我也不知道到底多久】我想着把重复的lncRNA实战的`代码全文`和`中间的数据`直接共享出来，方便大家学习RNA-seq各个步骤所需要的中间数据。

## 脚本依赖安装

其中r-base为4.1.3版本。需要注意这里使用的是`r-argparser` ，别装错了。

```sh
conda install -c conda-forge r-argparser #0.7
conda install -c conda-forge r-tidyverse
conda install -c bioconda bioconductor-edger #用了rpkm()函数算的FPKM
```

## 脚本使用

- `-i, --input_path`  指定含有featurecounts结果的文件夹
- `-p, --pattern` 用R中正则指定文件pattern。建议直接将featurecounts的结果文件后缀写为`.count` ，类似``<sample>.count` 格式，这个选项用默认的就可以了。

- `-o, --output_path` 输出文件结果文件夹指定
- `-f, --prefix ` 输出结果文件名前缀，默认为"`my`"

命令行：

```sh
./run_merge_fc_counts_normalization.R -i ./lncrna  -o ./
```

然后在当前目录有三个文件生成：`my_genes.counts`，`my_genes.fpkm`，`my_genes.tpm`

分别是基因的raw count矩阵，FPKM矩阵和TPM矩阵文件，所有结果文件均未进行基因筛选。

注意事项：

- 真不建议用`.count` 之外的后缀名做featurecounts的结果文件后缀，除非你想挑战下自己的软肋。

- TPM和FPKM的矩阵我没有设置保留的小数位数，有需要的自己`round()`一下吧。不算啥大毛病。
- 代码都在这里了，我就不直接分享脚本了。新手使用R脚本可能会有小报错，都是小问题，帮助你成长下。
- 特别注意：过程提示会有“`n  genes were not expressed in all samples!`” ，运行时`n`会是具体数值，这句提示的意思是有`n` 个基因在所有样本中的count值均为0。但是，这些基因并未过滤掉！

## 代码

说了半天废话，这才是主要的。

```R
#!/usr/bin/env Rscript
options(warn = -1)
suppressMessages(library(stringr))
suppressMessages(library(dplyr))
suppressMessages(library(tibble))
suppressMessages(library(edgeR))
suppressMessages(library(argparser)) #https://github.com/cran/argparser


# 参数设置
p <- arg_parser("Merge quantification files from featureCounts(linux version) and calculate FPKM/TPM")

p <- add_argument(p, "--input_path", help="input: a directory containing the counts matrix named with '<sample>.count'", type="character",default = "./")
p <- add_argument(p, "--pattern", help="limit pattern of input files using regular expression in R language",type="character",default = "*count$")
p <- add_argument(p, "--output_path", help="output: an existent directory", type="character",default = "./")
p <- add_argument(p, "--prefix", help="give the file of output matrix a prefix like '<output_prefix>_genes.*'", type="character",default = "my",short = "-f")

# 参数解析与定义
argv <- parse_args(p)

path <- argv$input_path
pattern <- argv$pattern
output_path <- argv$output_path
output_prefix <- argv$prefix

# 代码主体
file_name <- dir(path = path,pattern = pattern)
file <- paste0(path,"/",file_name)
# merge all count matrix
df <- read.table(file[1], header = T,comment.char = "#") %>% select(c(1,6,7))
colnames(df)[3] <- basename(file[1]) %>% str_remove("\\.\\w+$")
cat("1 count matrix has merged!\n")
for (i in 2:length(file)) {
  df_tmp <- read.table(file[i], header = T,comment.char = "#") %>% select(c(1,7))
  colnames(df_tmp)[2] <- basename(file[i]) %>% str_remove("\\.\\w+$")
  df <- df %>% full_join(df_tmp,by="Geneid")
  cat(i," count matrixs have merged!\n")
}
cat("Congratulations! All count matrixs have merged!\n")

count_mat <- df %>% select(-2) %>% tibble::column_to_rownames(var = "Geneid")
gene_length <- df %>% select(1,2) %>% tibble::column_to_rownames(var = "Geneid")
filter_count_mat <- count_mat[rowSums(count_mat)>0,]
cat(nrow(count_mat)-nrow(filter_count_mat)," genes were not expressed in all samples!\n")

# caculate FPKM
cat("Calculating the FPKM…\n")
fpkm <- rpkm(count_mat,gene.length = gene_length$Length) %>% as.data.frame()

# caculate TPM
cat("Calculating the TPM…\n")
fpkm2tpm <- function(fpkm){
  if((is.matrix(fpkm) | is.data.frame(fpkm)) & all(fpkm>=0)){ #fpkm所有值为非负且为矩阵或者数据框
    tpm <- t(t(fpkm)/colSums(fpkm))*10^6
  }else{
    stop("The fpkm must be a matrix or data.frame with nonnegative numerical values!")
  }
  return(tpm)
}
tpm <- fpkm2tpm(fpkm = fpkm) %>% as.data.frame() 

# write out 
cat("writing out raw counts matrix\n")
out_count_mat <- count_mat %>% rownames_to_column(var = "gene_id")
count_file <- paste0(output_path,output_prefix,"_","genes.counts")
write.table(out_count_mat, file = count_file,sep = "\t", col.names = TRUE, row.names = FALSE, quote = FALSE)

cat("writing out FPKM\n")
out_fpkm <- fpkm %>% rownames_to_column(var = "gene_id")
fpkm_file <- paste0(output_path,output_prefix,"_","genes.fpkm")
write.table(out_fpkm, file = fpkm_file,sep = "\t", col.names = TRUE, row.names = FALSE, quote = FALSE)

cat("writing out TPM\n")
out_tpm <- tpm %>% rownames_to_column(var = "gene_id")
tpm_file <- paste0(output_path,output_prefix,"_","genes.tpm")
write.table(tpm, file = tpm_file,sep = "\t", col.names = TRUE, row.names = FALSE, quote = FALSE)

cat("Congratulations! All of the missions have been completed!\n")

```

## 小结

本来台式机测试代码的时候读取处理什么的比较慢，写了一些提示。但是在服务器上运行“嗖嗖”的，这些提示好像有些多余了。

R包`argparser` 和`argparse` 都是R脚本中可用的参数传递用的R包。需要额外注意的是`argparse` 在2.0版本以后可以添加group命令，这样就可以实现在单个脚本中的多命令，也就可以实现，单个R脚本中，合并所有常见的定量软件的结果。

代码中加了一些判断，是为了以后封包练习的。但是还是稍显简陋。脚本的控制，状态返回值之类的都没有。没有看过专门的R编程书，暂时也用不到，也就没学没加。有相关资料的大家可以给我分享下，感激不尽~

其实，这个脚本完全可以用`Rsubread` 加上定量的内容，在一个脚本中实现定量、合并和标准化。