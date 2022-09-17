---
title: Uniprot数据库dat文件转换为fasta文件
tags:
  - perl
  - Uniprot
  - 数据库
keywords: 
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
#!/usr/bin/env perl
use strict;
#use warnings;
use File::Basename;
use Getopt::Std;
use POSIX; #计算时间

####################################################
# 
# Usage of dat2fa_swiss_prot.pl
# 
####################################################
sub usage{
    die(
        qq!
Usage:     perl $0 -i <dat file(required)> -o <fasta file>
Function:  to convert dat file from uniprot(Swiss-Prot) to fasta file.
Command:   -i input file (dat file) from uniprot(Swiss-Prot)(required). A gzip file with '.gz' suffix is also acceptable.
           -o output file with fasta format. default: related to input file. 
           -h get the help tips.
Author:    Shimao Zheng, zhengshimao007@163.com
version:   v0.1
Update:    2022.7.8
Notes:     Follow my WeChat public account 'The_Elder_Student'
\n! 
    )
}
#&usage; #调用帮助测试
my $dat_file;
my $fasta_file; 

####################################################
#
# 命令行参数的定义和获取，记录程序初始时间，设置参数默认值.
# Get the parameter and provide the usage.
#
####################################################
my %opts;
getopts( 'i:o:h', \%opts );
&usage && exit 1 unless ( exists $opts{i} );
&usage if ( exists $opts{h} );
my $start_time=time;
print strftime("Start time is %Y-%m-%d %H:%M:%S\n", localtime(time));
#output file
my ($name, $path, $suff)=fileparse("$opts{i}", qr/\.\w+$/);
if($suff eq "\.gz"){
    $fasta_file = "$name.gz";
}else{
    $fasta_file = "$name.$suff.fasta";
}

$dat_file = $opts{i};
$fasta_file = $opts{o}||"$opts{i}.fasta";

die "$fasta_file exist. Give another file name please!\n" if (-e $fasta_file);

print "Input file is $dat_file\nOutput file is $fasta_file \n\n";


####################################################
#
# To convert dat file from uniprot(Swiss-Prot)  to fasta file.
#
####################################################
if($suff eq "\.gz"){
    open DAT,"zcat $opts{i} | " || die "ERROR:can't open $opts{i} !";
}else{
    open DAT,"<","$opts{i}" || die "ERROR:can't open $opts{i} !";
}

open FA,">","$fasta_file" || die "ERROR:can't open $fasta_file!";
#number 44064;

    my $id; #UniProt ID
    my $ac; #accession
    my $sv; #sequence version
    my $full; #full name;
    my $os; #Organism
    my $ox; #Organism Taxonomy
    my $pe; #Protein Existence
    my $sq; #Sequence

$/="//\n"; # don't use '//',because http links are included in dat files;
while(<DAT>){

    my $gn; #gene name #必须命名在循环内，因为部分序列无GN。如果定义不在循环内，则会造成下一个模块内GN未被重置。
   
    chomp;
    #print ;

    #if(m/^ID\s+(\S+)\s+/){ $id = $1 }; #first method
    $id = $1 if(m/^ID\s+(\S+)\s+/); #second method

    $ac = $1 if(m/AC\s+(\w+);/); #Accession #部分含有多个ID，这里只选了第一个。
    #print "$ac\n";

    $sv = $1 if( m/DT.*sequence version\s+(\d+)/); #why can't use "^DT"?;
    #if(/DT.*sequence version\s+(\d+)/){print "$1\n"};

    $full = $1 if(m/DE\s+RecName: Full=(.*);/); #匹配的第一个DE，后面的DE有同样格式的，被忽略了。
    #去掉可能含有的{ECO.*}
    $full =~ s/\s+\{.*\}//g;#{}从5.22开始，正则中的{}要加转义符 #报错解决https://www.codercto.com/a/92293.html

    $os = $1 if(m/OS\s+(.*)\./);
    $os =~ s/\s+\(.*\)//; #NOTE:'\';

    $ox = $1 if(m/OX\s+NCBI_TaxID=(\d+)/); #数字之后不一定是 ';'，如：11S1_CARIL

    $gn = $1 if(m/GN\s+Name=(.*?);/); #部分序列无GN,如108_SOLLC; 部分序列还有多个GN; 部分GN含有多个 ';'; 
    #print "$gn\n"; #报错无法解决，去掉 'use warnings;' 才可以运行。但是部分可能为空值。
    #尝试写GN是否存在，如果不存在，$gn重置为空

    $pe = $1 if(m/PE\s+(\d+):/);

    #print "$id\$ac\t$sv\t$full\t$os\t$ox\t$gn\t$pe\n";

    # 打印fasta格式的第一行
#=pod
    if($gn){
        print FA ">sp|$ac|$id $full OS=$os OX=$ox GN=$gn PE=$pe SV=$sv";
    }else{
        print FA ">sp|$ac|$id $full OS=$os OX=$ox PE=$pe SV=$sv";
    }
#=cut
    

    if(m/SQ   SEQUENCE\s+.*;/){  #学习捕获中的$`, $&, $'的含义。
=pod
    $`表示匹配起始位置之前的字符串
    $&表示匹配的内容，即//内的内容
    $'表示匹配终结位置之后的内容
=cut 
        #$sq = $';
        #$sq =~ s/\s+//g; #记得加/g 进行全局替换。
        #赋值加替换，单行写法
        ($sq = $') =~ s/ //g;#记得加/g 进行全局替换，删除空格。 
        print FA "$sq";
    }
}
close DAT;
close FA;
####################################################
#
# Record the program running time!
# 输出程序运行时间
#
####################################################
my $duration_time=time-$start_time;
print strftime("End time is %Y-%m-%d %H:%M:%S\n", localtime(time));
print "This compute totally consumed $duration_time s\n";
print "Follow my WeChat public account 'The_Elder_Student'\n";
print "OK!\n";

```



## 验证方法

手动验证，由于[Taxonomic divisions](https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/taxonomic_divisions/) 下的文件序列均存在于`complete` 版本下的文件[uniprot_sprot.fasta.gz](https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_sprot.fasta.gz) 。所以，可以复制下你得到的fasta文件ID到`uniprot_sprot.fasta.gz` 中进行搜索核对对应序列。

目前只有这种方法，不够高明。

## 讨论

如果这个[Taxonomic divisions](https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/taxonomic_divisions/) 目录下的`uniprot_sprot*.dat.gz`  文件转换有问题，欢迎给我报错。
