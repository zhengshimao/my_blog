---
title: Uniprot数据库dat文件转换为fasta文件【已转为非公开，有需求请付费】
tags:
  - perl
  - Uniprot
  - 数据库
  - perl脚本
keywords: “数据库,注释”
categories:
 - 归档

date: 2022-09-17
  
toc: true
draft: false
url: post/uniprot_dat2fasta.html
---

# Uniprot数据库

## 关于序列存储

Uniprot数据库中数据库存储方式之一是我们常用的**fasta**格式，方式之二是**dat**文件格式，方式之三是**xml**格式。

fasta格式，我只找到了`complete` 版本下的文件[uniprot_sprot.fasta.gz](https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_sprot.fasta.gz)。但是[Taxonomic divisions](https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/taxonomic_divisions/)下的生物分类的文件中，没找到fasta格式。

（以下内容为我个人思考内容给大家简单介绍下我的需求，没见别人这么搞过，不知道对错，也不知道有无必要，所以，大家学学代码就可以了。）

由于我做的是**植物**，论文里也是想用Uniprot数据库做个注释，但我强迫症，我就只想用Uniprot中的植物序列信息做注释。所以，这里我写了个perl脚本把dat格式的[ uniprot_sprot_plants.dat.gz ](https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/taxonomic_divisions/uniprot_sprot_plants.dat.gz) （目前只有4万多条序列）  转换为fasta格式。其实本脚本也适用其它 `uniprot_sprot*.dat.gz`  文件。

下面介绍下FASTA序列信息、脚本使用方法、脚本内容、验证方法。

## 用到的目录链接

complete：https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/

Taxonomic divisions：https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/taxonomic_divisions/

## FASTA序列信息注解

https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_sprot.fasta.gz

```sh
>sp|P31237|ACCO_ACTDE 1-aminocyclopropane-1-carboxylate oxidase OS=Actinidia deliciosa OX=3627 GN=ACO PE=2 SV=1
MEAFPVIDMEKLNGEERAPTMEKIKDACENWGFFELVNHGISHELMDTVERLTKEHYNKC
MEQRFKEMVATKGLEAVQSEINDLDWESTFFLRHLPVSNISEIPDLEQDHRKAMKEFAEK
LEKLAEQLLDLLCENVGLEKGYLKKAFYGSKGPTFGTKVSNYPPCPRPELIKGLRAHTDA
GGIILLFQDNKVSGLQLLKDGEWIDVPPMKHSIVINIGDQLEVITNGKYKSVMHRVIAQP
DGNRMSIASFYNPGSDAVMYPAPALVDKEEDQQKQVYPKFVFEDYMKLYAGLKFQAKEPR
FEAMKAMENAVNLGPIATI
```

\> 后的注释信息进行简单说明。

- **sp**：Swiss-Prot数据库的简称，也就是上面说的验证后的蛋白数据库。

那Trembl数据库呢？未下载过，没看过。

- **P31237**：UniProt ID号   对应dat文件中的`AC  P31237;`  <u>dat文件中的编号可能有多个，并以`;` 分割，而在fasta序列中只包含第一个ID号。</u>

- **ACCO_ACTDE 1-aminocyclopropane-1-carboxylate oxidase**：蛋白质名称。`ACCO_ACTDE`对应dat文件中的`ID`，`1-aminocyclopropane-1-carboxylate oxidase`是dat文件中的`DE  RecName: Full=1-aminocyclopropane-1-carboxylate oxidase;`。

- **OS=Actinidia deliciosa** ：OS是Organism简称，Actinidia deliciosa是美味猕猴桃的拉丁文名称，说明该蛋白是来自美味猕猴桃。对应dat文件中的`OS ` 。
- **OX=3627**：Organism Taxonomy，也就是物种分类数据库Taxonom y ID。对应dat文件中的`OX  NCBI_TaxID=3627;`
- **GN=ACO** ：Gene name，基因名为ACO 。对应dat文件中的`GN  Name=ACO;` 。部分序列无`GN项`，如`108_SOLLC` (一种植物蛋白)。<u>部分蛋白中可能没有`GN` 项</u>。
- **PE=2** ：Protein Existence，蛋白质可靠性，对应5个数字，数字越小越可靠：对应dat文件中的`PE  2: Evidence at transcript level;` 
  1：Experimental evidence at protein level
  2：Experimental evidence at tranlevel
  3：Protein inferred from homology
  4：Protein predicted
  5：Protein uncertain
- **SV=1**：Sequence Version，序列版本号。对应dat文件中的`DT  01-JUL-1993, sequence version 1.` 

## 脚本使用方法

```sh
perl dat2fa_swiss_prot.pl -i uniprot_sprot_plants.dat.gz -o output.fasta
```

- -i 输入文件，是必需参数。允许解压后的dat文件，也允许gzip压缩的`.gz` 后缀文件。
- -o 指定输出文件。如果未指定，则根据输入文件进行定义，且输出路径与输入文件相同。如果结果文件已经存在，则无法运行，必须更换或者删除输出文件名称。

注意：脚本有部分内容（序列ID中的'sp|'）写死，只能用于[Taxonomic divisions](https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/taxonomic_divisions/) 下的uniprot_sprot文件`uniprot_sprot*.gz` ，不能用于 uniprot_trembl 文件。

## 脚本内容

保留了脚本内注释，也在学perl脚本的注意看一眼吧。

```perl
# 已转为非公开脚本
```

## 验证方法

手动验证，由于[Taxonomic divisions](https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/taxonomic_divisions/) 下的文件序列均存在于`complete` 版本下的文件[uniprot_sprot.fasta.gz](https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_sprot.fasta.gz) 。所以，可以复制下你得到的fasta文件ID到`uniprot_sprot.fasta.gz` 中进行搜索核对对应序列。

目前只有这种方法，不够高明。

## 讨论

如果这个[Taxonomic divisions](https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/taxonomic_divisions/) 目录下的`uniprot_sprot*.dat.gz`  文件转换有问题，欢迎给我报错。
