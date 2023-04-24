---
title: Read2Tree
tags:
  - 软件使用
  - linux
keywords: “软件使用,linux”
categories:
 - 软件使用

date: 2023-04-24
  
toc: true
draft: false
url: ability/read2tree.html
---

## 相关链接

github：https://github.com/DessimozLab/read2tree

作者：https://lab.dessimoz.org/blog/2023/04/23/read2tree-infers-trees-from-raw-reads-behind-the-paper

文章：[Inference of phylogenetic trees directly from raw sequencing reads using Read2Tree | Nature Biotechnology](https://www.nature.com/articles/s41587-023-01753-4)

## 安装

### conda安装【实际使用】

```sh
conda create -n r2t python=3.10
# condact r2t
conda install mamba
mamba install -c bioconda  read2tree
```

### docker安装

```sh
docker pull dessimozlab/read2tree:latest
```

### 源码安装

依赖用conda安装，read2tree从github安装

```sh
conda install -c conda-forge biopython numpy Cython ete3 lxml tqdm scipy pyparsing requests natsort pyyaml filelock
conda install -c bioconda dendropy pysam
conda install -c bioconda mafft iqtree ngmlr nextgenmap samtools

git clone https://github.com/DessimozLab/read2tree.git
cd read2tree
python setup.py install
```

## 输入文件

### 输入一：

测序的fastq文件

### 输入文件二：marker genes

步骤来源：https://github.com/DessimozLab/read2tree/wiki/obtaining-marker-genes

- 推荐从[OMA browser](https://omabrowser.org/oma/export_markers) 下载【是真的好慢啊！！！】：操作步骤见：https://github.com/DessimozLab/read2tree/wiki/obtaining-marker-genes

- 细菌或哺乳动物： We provide two sets of markergenes for Mammalia and Bacteria [here](https://github.com/DessimozLab/read2tree/tree/main/archive/set_marker_genes). 
- 病毒：操作说明见[obtaining marker genes for viral dataset](https://github.com/DessimozLab/read2tree/wiki/obtaining-marker-genes-for-viral-dataset)，就是 [OMA standalone](https://omabrowser.org/standalone/) 推断 marker genes

冠状病毒marker genes https://corona.omabrowser.org/oma/export_markers

- 从 [NCBI refSeq](https://www.ncbi.nlm.nih.gov/refseq/)下载一些列你感兴趣的进化分支的蛋白质和cDNA，然后使用 [OMA standalone](https://omabrowser.org/standalone/) 推断 marker genes。

## 运行

### 单物种模式

```sh
read2tree --tree --standalone_path marker_genes/ --reads read_1.fastq read_2.fastq  --output_path output
```

### 多物种模式

```sh
read2tree --standalone_path marker_genes/ --output_path output --reference  # this creates just the reference folder 01 - 03
read2tree --standalone_path marker_genes/ --output_path output --reads species1_R1.fastq species2_R2.fastq
read2tree --standalone_path marker_genes/ --output_path output --reads species2_R1.fastq species2_R2.fastq
read2tree --standalone_path marker_genes/ --output_path output --reads species3_R1.fastq species3_R2.fastq
read2tree --standalone_path marker_genes/ --output_path output --merge_all_mappings --tree
```

## 示例

使用github中自带的测试数据集`tests` 文件夹。

```sh
git clone https://github.com/DessimozLab/read2tree.git
cd read2tree/tests/
read2tree --threads 10 --tree --standalone_path ./marker_genes/ --reads ./sample_1.fastq ./sample_2.fastq --species_name sample1  --output_path ./
```

- --threads  mapping时的 线程数
- --tree 计算进化树，否则只输出比对结果。
- --standalone_path marker genes文件所在文件夹
- --reads 测序文件
- --species_name 所分析文件的物种名称，默认为reads文件名，**决定了后续nwk文件的名称和nwk文件内分支的名称**。
- --output_path  指定输出结果文件夹，**如果文件夹不存在，将创建**。

发育树文件`tree_sample1.nwk`

```sh
(sample1:0.0505889663,((MNELE:0.9367936021,XENLA:0.1449402450):0.1376429337,(HUMAN:0.0039311745,GORGO:0.0103980892):0.0495577264):0.0629576039,RATNO:0.0173433314);
```

所有结果文件：数字开头的为文件夹。

```sh
mplog.log
tree_sample_1.nwk
concat_sample_1_dna.phy
concat_sample_1_aa.phy
06_align_sample_1_dna/
06_align_sample_1_aa/
05_ogs_map_sample_1_dna/
05_ogs_map_sample_1_aa/
sample_1_all_cov.txt
sample_1_all_sc.txt
04_mapping_sample_1/
03_align_aa/
03_align_dna/
02_ref_dna/
01_ref_ogs_aa/
01_ref_ogs_dna/
```

## 软件详细参数

```sh
usage: read2tree [-h] [--version] [--output_path OUTPUT_PATH]
                 --standalone_path STANDALONE_PATH [--reads READS [READS ...]]
                 [--read_type READ_TYPE] [--threads THREADS] [--split_reads]
                 [--split_len SPLIT_LEN] [--split_overlap SPLIT_OVERLAP]
                 [--split_min_read_len SPLIT_MIN_READ_LEN] [--sample_reads]
                 [--genome_len GENOME_LEN] [--coverage COVERAGE]
                 [--min_cons_coverage MIN_CONS_COVERAGE]
                 [--dna_reference DNA_REFERENCE] [--sc_threshold SC_THRESHOLD]
                 [--ngmlr_parameters NGMLR_PARAMETERS] [--check_mate_pairing]
                 [--debug] [--sequence_selection_mode SEQUENCE_SELECTION_MODE]
                 [-s SPECIES_NAME] [--tree] [--merge_all_mappings] [-r]
                 [--min_species MIN_SPECIES] [--single_mapping SINGLE_MAPPING]
                 [--ref_folder REF_FOLDER]
                 [--remove_species_mapping REMOVE_SPECIES_MAPPING]
                 [--remove_species_ogs REMOVE_SPECIES_OGS] [--keep_all_ogs]
                 [--ignore_species IGNORE_SPECIES]

read2tree is a pipeline allowing to use read data in combination with an OMA
standalone output run to produce high quality trees.

optional arguments:
  -h, --help            show this help message and exit
  --version             Show programme's version number and exit.
  --output_path OUTPUT_PATH # 输出结果文件夹
                        [Default is current directory] Path to output
                        directory.
  --standalone_path STANDALONE_PATH # marker genes数据集
                        [Default is current directory] Path to the folder where marker genes
                        (i.e. reference orthologous groups) in fasta format are located.
  --reads READS [READS ...] # 测序数据
                        [Default is none] Reads to be mapped to reference. If
                        paired end add separated by space.
  --read_type READ_TYPE # reads 类型，长读长与短读长
                        [Default is "short" reads] Type of reads to use for
                        mapping, either "short" or "long". Either ngm for short reads or ngmlr for long
                        will be used.
  --threads THREADS     [Default is 1] Number of threads for the mapping using
                        ngm / ngmlr! # 线程数
  --split_reads         [Default is off] Splits reads as defined by split_len
                        (200) and split_overlap (0) parameters.
  --split_len SPLIT_LEN
                        [Default is 200] Parameter for selection of read split
                        length can only be used in combinationwith with long
                        read option.
  --split_overlap SPLIT_OVERLAP
                        [Default is 0] Reads are split with an overlap defined
                        by this argument.
  --split_min_read_len SPLIT_MIN_READ_LEN
                        [Default is 200] Reads longer than this value are cut
                        into smaller values as defined by --split_len.
  --sample_reads        [Default is off] Splits reads as defined by split_len
                        (200) and split_overlap (0) parameters.
  --genome_len GENOME_LEN
                        [Default is 2000000] Genome size in bp.
  --coverage COVERAGE   [Default is 10] coverage in X. Only considered if
                        --sample reads is selected.
  --min_cons_coverage MIN_CONS_COVERAGE
                        [Default is 1] Minimum number of nucleotides at
                        column.
  --dna_reference DNA_REFERENCE
                        [Default is None] Reference file that contains
                        nucleotide sequences (fasta, hdf5). If not given it
                        will usethe RESTapi and retrieve sequences from
                        http://omabrowser.org directly. NOTE: internet
                        connection required!
  --sc_threshold SC_THRESHOLD
                        [Default is 0.25; Range 0-1] Parameter for selection
                        of sequences from mapping by completeness compared to
                        its reference sequence (number of ACGT basepairs vs
                        length of sequence). By default, all sequences are
                        selected.
  --ngmlr_parameters NGMLR_PARAMETERS
                        [Default is none] In case this parameters need to be
                        changed all 3 values have to be changed [x,subread-
                        length,R]. The standard is: ont,256,0.25.
                        Possibilities for these parameter can be found in the
                        original documentation of ngmlr.
  --check_mate_pairing  Check whether in case of paired end reads we have
                        consistent mate pairing. Setting this option will
                        automatically select the overlapping reads and do not
                        consider single reads.
  --debug               [Default is false] Changes to debug mode: * bam files
                        are saved!* reads are saved by mapping to OG
  --sequence_selection_mode SEQUENCE_SELECTION_MODE
                        [Default is sc] Possibilities are cov and cov_sc for
                        mapped sequence.
  -s SPECIES_NAME, --species_name SPECIES_NAME
                        [Default is name of read 1st file] Name of species for
                        mapped sequence.
  --tree                [Default is false] Compute tree, otherwise just output
                        concatenated alignment!
  --merge_all_mappings  [Default is off] In case multiple species were mapped
                        to the same reference this allows to merge this
                        mappings and build a tree with all included species!
  -r, --reference       [Default is off] Just generate the reference dataset
                        for mapping.
  --min_species MIN_SPECIES
                        Min number of species in selected orthologous groups.
                        If not selected it will be estimated such that around
                        1000 OGs are available.
  --single_mapping SINGLE_MAPPING
                        [Default is none] Single species file allowing to map
                        in a job array.
  --ref_folder REF_FOLDER
                        [Default is none] Folder containing reference files
                        with sequences sorted by species.
  --remove_species_mapping REMOVE_SPECIES_MAPPING
                        [Default is none] Remove species present in data set
                        after mapping step completed and only do analysis on
                        subset. Input is comma separated list without spaces,
                        e.g. XXX,YYY,AAA.
  --remove_species_ogs REMOVE_SPECIES_OGS
                        [Default is none] Remove species present in data set
                        after mapping step completed to build OGs. Input is
                        comma separated list without spaces, e.g. XXX,YYY,AAA.
  --keep_all_ogs        [Default is on] Keep all orthologs after addition of
                        mapped seq, which means also the OGs that have no
                        mapped sequence. Otherwise only OGs are used that have
                        the mapped sequence for alignment and tree inference.
  --ignore_species IGNORE_SPECIES
                        [Default is none] Ignores species part of the OMA
                        standalone pipeline. Input is comma separated list
                        without spaces, e.g. XXX,YYY,AAA.
```

## 其它

数据库源文件蛋白序列文件`oma-seqs.fa.gz` 中，`>` 后面紧接的是空格。