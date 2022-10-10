---
title: 封个Bash脚本 | 脚本根据Project 编号获取3.5种方式从ENA获取数据的脚本与执行命令
tags:
  - bash脚本
  - linux
  - 转录组
  - 数据上传下载
keywords: “数据库,注释”
categories:
 - 归档

date: 2022-10-05
  
toc: true
draft: false
url: post/run_get_bioinfo_sample.sh.html
---

# 封个Bash脚本 | 脚本根据Project 编号获取3.5种方式从ENA获取数据的脚本与执行命令

## 脚本介绍与使用

3.5种？因为第3种下载方式给了2种`SRA`文件下载方式。

本文用到的有效脚本是从[ENA](https://www.ebi.ac.uk/ena/browser/search) 获取的有效信息，我已经放弃SRA了。

每种方式都包含了`数据下载` 、 `数据校验` 和`重命名`等命令，最终的结果文件都是`fastq.gz`。

关于 `run_rename_sample.sh` 的脚本，由于上传数据的人什么奇葩都有，这一步中的脚本必须在运行前进行检查是否符合我们常用的命名规范以及是否能按照区分不同样品。主要是改个名字，做下游分析方便点，不至于眼花。

适用于`单端测序`与`双端测序`。

其实严格来说，后两种方式`kingfisher`作者都封在了软件内。后两种只做未安装`kingfisher` 用户的备用方法。

由于所有脚本的输入、和输出都是`在当前目录` 中，所以，屏幕提示的3种下载方式为`主体命令`，需要你在使用时根据需要进行简单修改。

脚本内所用数据`PRJNA881764` 是一个miRNA的单端测序项目，6个数据文件。`PRJNA275632`是一个猪的普通转录组双端测序项目。

`grabseqs` 下载数据方式并未入到本文中。因为之前探索过一次，失败了！

## 脚本依赖

每个方式单独说依赖，不需要全部安装，根据自己选择的方式进行下载。

第一种方式：`kingfisher`  【我一直在用的下载方法】

第二种方式：`ascp`

第三种方式：`axel` 、`sra-tools` 、 `parallel-fastq-dump` 和`pigz`

屏幕输出的代码中还涉及到了`ParaFly` ，不装也行，直接`bash ` 相应脚本，就是慢点而已。

## 脚本使用

项目测序编号`PRJNA881764`

```sh
bash run_get_bioinfo_sample.sh PRJNA881764 #建议将屏幕输出重定向到一个文件内。屏幕输出即为你运行所产生bash脚本的主体命令
```

不知道怎么用了就运行

```sh
bash run_get_bioinfo_sample.sh
```

会获取如下提示：

```sh
Usage:bash run_get_bioinfo_sample.sh PRJNA881764
please give a parameter of project number,e.g. PRJNA275632
```

正确运行会获取一个`ena_info_sample`文件夹

```sh
tree ena_info_sample/
ena_info_sample/
├── fastq_md5.txt
├── progect_infor.txt
├── run_axel_sra_from_ena.sh
├── run_downloaded_fastq_using_aspera.sh
├── run_gzip_parallel-fastq-dump.sh
├── run_pigz_fasterq_dump.sh
├── run_rename_sample.sh
├── run_wget_sra_from_ena.sh
└── sra_md5.txt
```

还会有屏幕提示三种下载方式，如下：

```sh
###################################################
#The 1st method
conda activate kingfisher
kingfisher get -p PRJNA881764 -m ena-ftp aws-http aws-cp prefetch --download-threads 8 -f fastq.gz
md5sum -c fastq_md5.txt > md5.res
#在进行下一步之前先检查重命名后的名字是否规范,有的项目信息有奇葩
bash run_rename_sample.sh

###################################################
#The 2nd method

ParaFly -c run_downloaded_fastq_using_aspera.sh -CPU 5 -failed_cmds failed_run_downloaded_fastq_using_aspera.txt
md5sum -c fastq_md5.txt > md5.res
#在进行下一步之前先检查重命名后的名字是否规范,有的项目信息有奇葩
bash run_rename_sample.sh

###################################################
#The 3rd method
ParaFly -c run_wget_sra_from_ena.sh -CPU 10 -failed_cmds failed_wget_sra_from_ena.txt #wget下载sra
ParaFly -c run_axel_sra_from_ena.sh -CPU 1 -failed_cmds failed_axel_sra_from_ena.txt #axel下载sra #设置多CPU可能报错,原因未知
md5sum -c sra_md5.txt > md5.res #校验
ParaFly -c run_pigz_fasterq_dump.sh -CPU 2 -failed_cmds failed_pigz_fasterq_dump.txt #首选
ParaFly -c run_gzip_parallel-fastq-dump.sh -CPU 5 -failed_cmds failed_gzip_parallel-fastq-dump.txt #次选
#在进行下一步之前先检查重命名后的名字是否规范,有的项目信息有奇葩
bash run_rename_sample.sh

```

## 主体代码

```sh
#!/usr/bin/env bash

#检查$1 # $#表示参数格式  $?表示上一个命令的返回状态
if [ $# -eq 0 ];then 
	echo -e "Usage:bash $0 PRJNA881764\nplease give a parameter of project number,e.g. PRJNA275632";
	exit 1;
fi
echo $1 |grep -q 'PRJNA'
if [ $? -ne 0 ];then 
	echo -e "Usage:bash $0 PRJNA881764\nplease give a parameter of project number,e.g. PRJNA275632";
	exit 1;
fi

mkdir -p ena_info_sample && cd ena_info_sample

project=$1 #PRJNA275632
url="https://www.ebi.ac.uk/ena/portal/api/filereport?accession=$project&result=read_run&fields=study_accession,sample_accession,experiment_accession,run_accession,tax_id,scientific_name,fastq_md5,fastq_ftp,fastq_aspera,submitted_ftp,sra_md5,sra_ftp,sample_title&format=tsv&download=true&limit=0"
if [ -f progect_infor.txt ];then
	next
else
	wget  "$url" -O progect_infor.txt -q;
fi

if [ $? -eq 0 ];then
    echo "Successfully! progect_infor.txt downloaded!";
else
    echo "ERROR:Download of the progect_infor.txt failed!"
    exit 1;
fi

# rename sample name #处理空格部分略显啰嗦了2022.10.7
sed '1d' progect_infor.txt| awk -F "\t" '{print "rename s/"$4"/"$13"/g "$4"*.gz"}' |sed -e "s/rename s/rename 's/g" -e "s/\/g/\/g'/g" -e 's/-/_/g' -e 's/ /_/g' |sed -e 's/rename_/rename /g' -e 's/_SRR/ SRR/g' > run_rename_sample.sh

# get md5 of fastq
cat progect_infor.txt |awk -F"\t"  '{print $7}' |sed 's/;/\n/g' > fq_md5
cat progect_infor.txt |awk -F"\t"  '{print $8}' |sed 's/;/\n/g'  | sed 's/ftp.*SRR/SRR/g' > fq_file
paste -d "  " fq_md5 fq_file |sed '1d'> fastq_md5.txt
rm fq_file fq_md5

# run_downloaded_fastq_using_aspera.sh
cat progect_infor.txt |awk -F "\t" '{print $9 }'| sed -e '1d' -e 's/;/\n/g' |awk '{print "ascp  -vQT -l 500m -P33001 -k 1 -i ~/.aspera/connect/etc/asperaweb_id_dsa.openssh era-fasp@"$1" ./"}' > run_downloaded_fastq_using_aspera.sh
# example
# ascp  -vQT -l 500m -P33001 -k 1 -i ~/.aspera/connect/etc/asperaweb_id_dsa.openssh era-fasp@fasp.sra.ebi.ac.uk:/vol1/fastq/SRR122/079/SRR12207279/SRR12207279_1.fastq.gz  ./

# run_downloaded_sra_using_wget|axel.sh

cat progect_infor.txt |sed '1d'|awk -F "\t" '{print "wget -c -O "$4".sra "$12 }' > run_wget_sra_from_ena.sh
cat progect_infor.txt |sed '1d'|awk -F "\t" '{print "axel -n10 -o "$4".sra "$12 }' > run_axel_sra_from_ena.sh

# get md5 of sra
cat progect_infor.txt |sed '1d'|awk -F "\t" '{print $11"  "$4".sra"}' >sra_md5.txt

# fasterq-dump + pigz #首选
cat progect_infor.txt |sed '1d'|awk -F "\t" '{print "fasterq-dump --threads 8 --split-3 -O ./ "$4".sra;pigz -p 8 "$4"*.fastq"}' > run_pigz_fasterq_dump.sh #328837
# parallel-fastq-dump不加--gzip是未压缩的fastq文件,太占空间，舍
#cat progect_infor.txt |sed '1d'|awk -F "\t" '{print "parallel-fastq-dump --threads 8 --outdir ./ --split-3 -s "$4".sra"}' > parallel-fastq-dump.sh #280122
# parallel-fastq-dump的--gzip模式很慢
cat progect_infor.txt |sed '1d'|awk -F "\t" '{print "parallel-fastq-dump --threads 8 --outdir ./ --split-3  --gzip -s "$4".sra"}' > run_gzip_parallel-fastq-dump.sh #282962

# 赋予所有bash脚本可执行权限
chmod u+x *.sh
# 回到原文件夹
cd ..
# 赋予该bash脚本权限
chmod u+x $0


# 常用Kingfisher下载
echo "###################################################"
echo -e "#The 1st way"
echo "conda activate kingfisher"
echo "kingfisher get -p $1 -m ena-ftp aws-http aws-cp prefetch --download-threads 8 -f fastq.gz"
echo "md5sum -c fastq_md5.txt > md5.res"
echo "#在进行下一步之前先检查重命名后的名字是否规范,有的项目信息有奇葩"
echo "bash run_rename_sample.sh"
echo
# ascp从ENA下载fastq文件
echo "###################################################"
echo -e "#The 2nd way"
echo "ParaFly -c run_downloaded_fastq_using_aspera.sh -CPU 5 -failed_cmds failed_run_downloaded_fastq_using_aspera.txt"
echo "md5sum -c fastq_md5.txt > md5.res"
echo "#在进行下一步之前先检查重命名后的名字是否规范,有的项目信息有奇葩"
echo "bash run_rename_sample.sh"
echo
# ascp从ENA下载sra文件 #使用axel或者wget
echo "###################################################"
echo -e "#The 3rd way"
echo "ParaFly -c run_wget_sra_from_ena.sh -CPU 10 -failed_cmds failed_wget_sra_from_ena.txt #wget下载sra"
echo "ParaFly -c run_axel_sra_from_ena.sh -CPU 1 -failed_cmds failed_axel_sra_from_ena.txt #axel下载sra #设置多CPU可能报错,原因未知"
echo "md5sum -c sra_md5.txt > md5.res #校验"
echo "bash rename_sample.sh"
echo "ParaFly -c run_pigz_fasterq_dump.sh -CPU 2 -failed_cmds failed_pigz_fasterq_dump.txt #首选"
parallel-fastq-dump -h 1>/dev/null
if [ $? -eq 0 ];then
    echo 'ParaFly -c run_gzip_parallel-fastq-dump.sh -CPU 5 -failed_cmds failed_gzip_parallel-fastq-dump.txt #次选'
else
    echo "conda install -c bioconda parallel-fastq-dump"
    echo 'ParaFly -c run_gzip_parallel-fastq-dump.sh -CPU 5 -failed_cmds failed_gzip_parallel-fastq-dump #次选'
fi
echo "#在进行下一步之前先检查重命名后的名字是否规范,有的项目信息有奇葩"
echo "bash run_rename_sample.sh"
exit 0

```

## 探索SRA的无效代码

```sh
###############################################
# 无结果的探索
###############################################
# 使用esearch 和 efetch从SRA获取信息后下载SRA。
# 例子：miRNA的单端测序文件 PRJNA881764
# 存在的问题
# fastq-dump可以分开下载的SRR21619724.lite.1文件为fastq，但是fasterq-dump不行
# SRA信息表中的链接通过axel与wget下载，文件大小相同（但远小于应有值），wget与axel下载结果相同，md5sum校验的结果也一致！
# SRR21619724.lite.1通不过SRA下载的信息第46列`RunHash`的校验，也无法通过上方ENA下载信息中的sra_md5校验。且两个信息表中的sra校验信息不同。

:<<EOF
mkdir -p sra_info_sample && cd sra_info_sample
conda install -c bioconda entrez-direct
esearch -db sra -query PRJNA275632 | efetch -format runinfo > sra_info.csv
# download sra # $1,$10,$47  Run, download_path, RunHash
cat sra_info.csv |sed '1d'|awk -F ',' '{print "axel -n20 "$10}' > sra_axel_download.sh
cat sra_info.csv |sed '1d'|awk -F ',' '{print "wget -c "$10}' > sra_wget_download.sh
# RunHash of sra #不知道如何校验⭐
#cat sra_info.csv |sed '1d'|awk -F ',' '{print $46"  "$1".sralite.1"}' >sra.runhash

# fasterq-dump
cat sra_info.csv |sed '1d'|awk -F ',' '{print "fasterq-dump --threads 6 --split-3 -O ./ "$1".sralite.1"}'

# gzip fastq
num=`cat sra_info.csv |sed '1d' |wc -l` #总样本数
# 单线程gzip
if [ -f gzip_sra_fq.sh ];then rm gzip_sra_fq.sh;fi

for i in `seq $num`;do
    #echo $i
    line=`cat sra_info.csv |sed '1d'|sed -n "$i"p`
    LibraryLayout=`echo $line | awk -F ',' '{print $16}'`
    if [ $LibraryLayout == "PAIRED" ];then #PAIRED加不加引号均可？
        echo '$line' | awk -F ',' '{print "gzip "$1"_1.fastq;gzip "$1"_2.fastq;"}' >> gzip_sra_fq.sh
    elsif [ $LibraryLayout == "SINGLE" ];then #SINGLE
        echo '$line' | awk -F ',' '{print "gzip "$1"_1.fastq;"}' >> gzip_sra_fq.sh
    fi
done
# 多线程pigz
if [ -f pigz_sra_fq.sh ];then rm pigz_sra_fq.sh;fi

for i in `seq $num`;do
    #echo $i
    line=`cat sra_info.csv |sed '1d'|sed -n "$i"p`
    LibraryLayout=`echo $line | awk -F ',' '{print $16}'`
    if [ $LibraryLayout == "PAIRED" ];then #PAIRED加不加引号均可？
        echo '$line' | awk -F ',' '{print "gzip "$1"_1.fastq;gzip "$1"_2.fastq;"}' >> pigz_sra_fq.sh
    elsif [ $LibraryLayout == "SINGLE" ];then #SINGLE
        echo '$line' | awk -F ',' '{print "gzip "$1"_1.fastq;"}' >> pigz_sra_fq.sh
    fi
done

cd ..
EOF

```

## 小结

好家伙，今天是复习linux基础知识点的一天。

小细节是真的多！

写脚本、比较命令运行速度和测试脚本一天没吃饭，最后还把它免费分享出来，我是为了什么？得好好想一下。。