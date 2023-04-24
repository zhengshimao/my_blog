---
title:  RNA-seq定量软件使用示例与经验
tags:
  - 软件使用
  - 转录组
keywords: “软件使用,linux”
categories:
 - 软件使用

date: 2023-04-23
  
toc: true
draft: false
url: post/rna_seq_quant.html
---

RNA-seq定量软件有`featureCounts` , `htseq-count` ,`kallisto` , `rsem`, `salmon`, `stringtie`

还有个`卖鱼`的软件`Sailfish`，使用报错，github下多人向作者报错无人解答。我放弃了这个`卖鱼` 的。

## 代码

### featureCounts

基于基因水平`-g gene_id`定量

- 输入：比对到基因组的bam文件，基因组的gtf文件。

- 4分钟定量36个样品。

```R
mkdir -p featureCounts/fc_quant && cd featureCounts

id=`ls ../../3.mapping/*.bam |sed -e 's/.*\///g' -e 's/.bam$//g'`
bam_dir=../../3.mapping/
gtf=../../ref/Sus_scrofa.Sscrofa11.1.107.gtf
output_dir=fc_quant
for i in ${id};do
echo "featureCounts -T 10 -a ${gtf} -o ${output_dir}/${i}.count -p -B -t exon -g gene_id  ${bam_dir}/${i}.bam > ${output_dir}/${i}.log 2>&1 " >>  run_fc_counts.sh 
done

nohup ParaFly -c run_fc_counts.sh -CPU 4 -failed_cmds failed_fc_counts.log &
#3203366
```

### htseq-count

- 输入：比对到基因组的bam, 基因组的gtf文件

- 指定单线程，后面并行设置少了
- 太慢，耗时太久，无特殊要求不要用。
- `-n 10` 并行是在多bam文件时生效，每个文件仍然是单CPU。可以在上游`拆分bam文件`然后在此处指定多线程。
- `--with-header` 建议去掉。只会给第二列加个header，并且header为带指定路径的bam文件，建议你做的时候去掉，将文件名以样本名称命名，后期处理可能会更舒服些。
- `-c ` 选项指定输出文件会有问题，建议`--quiet` 【压制日志报告输出】 ，然后将输出到`标准输出`的结果重定向到文件中。不确定的是：此时好像不可以加`2>&1` ，猜测作者是指定`--quiet` 时将日志转为了`错误输出`。软件太慢，猜测未过多尝试。

```sh
mkdir -p htseq-count/htseq && cd htseq-count
#~/workspace/pig/11.quant/htseq-count
id=`ls ../../3.mapping/*.bam |sed -e 's/.*\///g' -e 's/.bam$//g'`
bam_dir=../../3.mapping/
gtf=../../ref/Sus_scrofa.Sscrofa11.1.107.gtf
output_dir=htseq
for i in ${id};do
echo "htseq-count -f  bam -m union --quiet -s yes --with-header    ${bam_dir}/${i}.bam  ${gtf} > ${output_dir}/${i}.count" >> run_htseq_count.sh 
done
# 单个文件运行
nohup ParaFly -c run_htseq_count.sh -CPU 15 -failed_cmds failed_htseq_count.log &
# 日志写入了nohup.out，并没有写入`${i}.count` 文件。
# 630900
# 2022.10.20
htseq-count -f bam -m union -s yes -c htseq/Sample_114_BF.count ../../3.mapping//Sample_114_BF.bam  ../../ref/Sus_scrofa.Sscrofa11.1.107.gtf >htseq/Sample_114_BF.log 2>&1

```

### Salmon

#### 基于reads的定量

注意事项：

- 需要建立index ，结果为一个文件夹
- 建立index输入为转录本序列，可以用`gffread -w` 提取。
- 定量时 `salmon quant -i` 指定上方的索引文件夹即可。
- 定量时 `salmon quant -o` 最后一级文件夹必须指定为`变量` ，否则会覆盖。建议写为类似`salmon_reads/${i}` ，自动新建文件夹。
- 构建索引输入：转录本序列；
- 定量时输入：构建的索引，reads文件。

```R
# ~/workspace/pig/11.quant/salmon
salmon quant --help-reads 
# 提转录本
gffread ../../ref/Sus_scrofa.Sscrofa11.1.107.gtf -g ../../ref/Sus_scrofa.Sscrofa11.1.dna.toplevel.fa -w ./pig.transcripts.fa 
# 构建索引
nohup salmon index  --threads 10 -k 31 -t pig.transcripts.fa -i pig.transcripts > log.salmon.index 2>&1 &
# 新建了一个索引文件夹pig.transcripts
$ ls pig.transcripts
# complete_ref_lens.bin   info.json         rank.bin             refseq.bin
# ctable.bin              mphf.bin          refAccumLengths.bin  seq.bin
# ctg_offsets.bin         pos.bin           ref_indexing.log     versionInfo.json
# duplicate_clusters.tsv  pre_indexing.log  reflengths.bin

mkdir -p salmon_reads
id=`ls ../../2.clean_reads/*_1.fastp.fq.gz |sed -e 's/.*\///g' -e 's/_1.fastp.fq.gz//g'`
for i in ${id};do
echo "salmon quant -i pig.transcripts -l A --gcBias --threads 10  -1 ../../2.clean_reads/${i}_1.fastp.fq.gz -2 ../../2.clean_reads/${i}_2.fastp.fq.gz -o salmon_reads/${i} > salmon_reads/${i}.log 2>&1 " >> run_salmon_reads.sh 
done #输出文件夹最后一级必须为变量，否则会覆盖

nohup ParaFly -c run_salmon_reads.sh -CPU 4 -failed_cmds failed_salmon_reads.log &
# 1936809
```

```sh
-l：--libType，测序文库类型，一般不知道什么文库的话用参数 A 让软件自动检测
#I = inward
#O = outward
#M = matching
#S = stranded
#U = unstranded
#F = read 1 (or single-end read) comes from the forward strand
#R = read 1 (or single-end read) comes from the reverse strand
#A = automatically determine
```

#### 基于比对定量

比对到基因组上的序列直接定量会报错，解决办法参考文档Quantifying in alignment-based mode中Note部分的3个方法解决。

可以重新用转录本序列比对定量，我用的服务器空间不足，这种方式未做。

### kallisto

- 对转录本建立索引，建立的索引是`单个文件`（建议名称中给`index` 等词作明显标记），不支持多线程，耗时7分钟
- 建立索引时：支持gzip压缩格式的转录本文件。
- 定量14分钟，36个样品。
- 定量时 `kallisto quant -o` 最后一级文件夹必须指定为`变量` ，否则会覆盖。建议写为类似`kallisto_quant/${i}` ，自动新建文件夹。
- 构建索引输入：转录本序列；
- 定量时输入：reads文件和索引

```sh
mkdir -p kallisto && cd kallisto
#~/workspace/pig/11.quant/kallisto
ln -s ../salmon/pig.transcripts.fa  ./
nohup kallisto index -k 31 -i pig.transcripts pig.transcripts.fa > index.log 2>&1 &
# 1283498
mkdir -p kallisto_quant
id=`ls ../../2.clean_reads/Sample*_1.fastp.fq.gz | sed -e 's/.*\///g' -e 's/_1.fastp.fq.gz$//g'`

for i in ${id};do
echo "kallisto quant --bias --threads 10 -i pig.transcripts  -o kallisto_quant/${i} ../../2.clean_reads/${i}_1.fastp.fq.gz ../../2.clean_reads/${i}_2.fastp.fq.gz >kallisto_quant/${i}.log 2>&1" >>run_kallisto_quant.sh
done
nohup ParaFly -c run_kallisto_quant.sh -CPU 4 -failed_cmds failed_kallisto_quant &

```

### RSEM

- 构建索引，需要单独指定一个文件夹
- 可以指定基因组`gemome.fa`和`gtf` 文件为定量构建索引
- `rsem-calculate-expression` 可以指定bam文件进行定量，但是也必须是基因转录本比对的结果，不能用基于基因组的比对结果。
- 本次比对使用bowtie2
- 此次bowtie2索引输入：基因组`gemome.fa`和`gtf` 文件
- 此次bowtie2比对定量输入：索引，reads文件【该软件也支持比对到转录本的bam文件作为输入定量。】

```shell
mkdir -p rsem/counts && cd rsem
mkdir -p index
#~/workspace/pig/11.quant/rsem
genome=../../ref/Sus_scrofa.Sscrofa11.1.dna.toplevel.fa
gtf=../../ref/Sus_scrofa.Sscrofa11.1.107.gtf
id=`ls ../../2.clean_reads/*1.fastp.fq.gz |sed -e 's/.*\///g' -e 's/_1.fastp.fq.gz//g'`
output_dir=counts
 
# 构建索引
nohup rsem-prepare-reference -p 20 --bowtie2 --gtf ${gtf}    ${genome}  ./index/pig_genome > rsem-prepare-reference.log 2>&1 & 
 
# 定量 #相当于重新用bowtie2比对，但是没有生成bam文件，因为这里只是为了生成计数矩阵，且我账号没空间了……
for i in ${id};do
echo "rsem-calculate-expression -p 8 --paired-end --bowtie2 --no-bam-output \
--bowtie2-sensitivity-level sensitive   \
../../2.clean_reads/${i}_1.fastp.fq.gz \
../../2.clean_reads/${i}_1.fastp.fq.gz  \
./index/pig_genome ./${output_dir}/${i} > ./${output_dir}/${i}.log 2>&1" >> run_rsem_genome.sh #脚本中为单行
done
# 后三项依次为 reads文件，索引和样本名称
# --bowtie2-sensitivity-level sensitive 为默认
nohup ParaFly -c run_rsem_genome.sh  -CPU 4 -failed_cmds failed_rsem_genome.log &

```

### Stringtie的prepDE.py

#### 使用stringtie -e 进行转录本定量

使用`stringtie -e` ，以基因组原`gtf文件`为参考进行定量,每个文件最后有`FPKM`和`TPM`。

```shell
mkdir -p stringtie/abundance_transcripts && cd stringtie
mkdir -p prepde
id=`ls ../../3.mapping/*.bam |sed -e 's/.*\///g' -e 's/.bam$//g'`
gtf=../../ref/Sus_scrofa.Sscrofa11.1.107.gtf
for i in ${id}; do
echo "stringtie -e -p 10 -G ${gtf}  -o  ./abundance_transcripts/${i}.matrix ../../3.mapping/${i}.bam >  abundance_transcripts/${i}.log 2>&1" >> run_stringtie_e.sh
done
nohup ParaFly -c run_stringtie_e.sh -CPU 4 -failed_cmds failed_stringtie_e.log &
```

准备`-i` 参数文件`sample_list.txt` ，第一列为样本名称，第二列为gtf文件，用`\t` 分割。

```shell
ids=`ls abundance_transcripts/Sample_*matrix`
for i in ${ids};do
sample=`echo ${i} | sed -e  's/.*\///g' -e 's/.matrix$//g'`
echo -e "${sample}\t${i}" >> sample_list.txt
done
```

#### 获取基因和转录本水平的count矩阵

根据上方的文件获取`基因`与`转录本`水平的`count`矩阵

```shell
prepDE.py -i sample_list.txt  -g prepde/gene_count_matrix.csv  -t prepde/transcript_count_matrix.csv
```