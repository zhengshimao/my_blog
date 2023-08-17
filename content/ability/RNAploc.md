---
title: 植物lncRNA鉴定之RNAplonc（2019年发表）
tags:
  - 转录组
  - lncRNA-seq
  - 软件使用
  - lncRNA鉴定
keywords: “转录组,lncRNA-seq”
categories:
 - 软件使用

date: 2022-11-16
  
toc: true
draft: false
url: ability/rnaploc.html
---

#  植物lncRNA鉴定之RNAplonc（2019年发表）

## 文章说明

英文：Pattern recognition analysis on long noncoding RNAs: a tool for prediction in plants

期刊：Brief Bioinform (IF: 8.99; Q1)

发表：2019.3.25

记录：2022.11.6

DOI： 10.1093/bib/bby034

引用：Negri TDC, Alves WAL, Bugatti PH, Saito PTM, Domingues DS, Paschoal AR. Pattern recognition analysis on long noncoding RNAs: a tool for prediction in plants. Brief Bioinform. 2019 Mar 25;20(2):682-689. doi: 10.1093/bib/bby034. PMID: 29697740.

地址：https://academic.oup.com/bib/article/20/2/682/4985385?login=false

" Considering this, we present a machine learning analysis and a classifier approach called RNAplonc (https://github.com/TatianneNegri/RNAplonc/) to identify lncRNAs in plants."

## 软件

- 软件地址： https://github.com/TatianneNegri/RNAplonc/ or http://rnaplonc.cp.utfpr.edu.br/
- 手册：https://github.com/TatianneNegri/RNAplonc/blob/master/manual.pdf

## 依赖与软件安装

依赖：txCDSpredict, CD-HIT-EST, Java, perl, python3

txCdsPredict: the precompiled binary is included in the package. Here is the link to download the source code:

https://github.com/ENCODE-DCC/kentUtils

CD-HIT-EST: https://github.com/weizhongli/cdhit/releases

perl：`bioperl` 

python3：`pandas ` `argparse  `  `numpy`  算是常见包，如果最后一步运行报错，自行安装。经测试`argparse  `  用`r-argparse`即可。

其实，`RNAplonc/RNAplonc` 有单个可用的 `txCdsPredict`文件。不必去github安装`kentUtils`

```sh
git clone https://github.com/TatianneNegri/RNAplonc.git
cd RNAplonc/RNAplonc
chmod +x FilterResults.py
chmod +x *.pl
chmod +x txCdsPredict
sed -i 's/#!\/usr\/bin\/perl/#!\/usr\/bin\/env perl/g' *.pl #修改下调用的perl，方便调用我们conda安装的模块。
# FilterResults.py 中作者用的`#!/usr/bin/env python3`

# 当前文件夹文件。留意RNAplonc.model文件！！！
# 200nt.pl  feature_extraction.pl  FilterResults.py  random_selection.pl  README.txt  RNAplonc.model  seq_test  txCdsPredict  weka.jar
pwd
# /home/data/vip13t28/biosoft/RNAplonc/RNAplonc
# 添加环境变量
conda install -c bioconda cd-hit
conda install -c bioconda perl-bioperl #feature_extraction.pl的依赖

```

## 使用

所有的结果后缀，都不要乱改！！！

软件手册中`step5`结果后缀写的是`ar` ，不要用，按我写的用`arff` 做后缀。

### step1. 准备fasta格式数据

将自带测试数据`seq_test/Citrus_sinensis_lncRNA_GREENC.fasta`重命名为`test.fa`

### step2. 200nt.pl 筛选长度

测试时必须全路径调用，否则报错。

输入：`test.fa`

输出：`test_.fasta`  #脚本内结果命名稍有瑕疵，但不影响结果

```sh
$ perl /home/data/vip13t28/biosoft/RNAplonc/RNAplonc/200nt.pl test.fa
$ ls
test.fa  test_.fasta #这个结果文件名称……
```

### step3. cd-hit-est（可选步骤）

输入：`test_.fasta`

输出：`cd-hit-est0.8.fasta`  `cd-hit-est0.8.fasta.clstr`

```sh
$ cd-hit-est -c 0.8 -T 5 -i test_.fasta  -o cd-hit-est0.8.fasta 
$ ls
cd-hit-est0.8.fasta  cd-hit-est0.8.fasta.clstr  test.fa  test_.fasta
```

### step4. txCdsPredict

输入：`cd-hit-est0.8.fasta`

输出：`cd-hit-est0.8.cds`

```sh
$ /home/data/vip13t28/biosoft/RNAplonc/RNAplonc/txCdsPredict cd-hit-est0.8.fasta cd-hit-est0.8.cds
$ ls
cd-hit-est0.8.cds  cd-hit-est0.8.fasta  cd-hit-est0.8.fasta.clstr  test.fa  test_.fasta
```

`cd-hit-est0.8.cds` 包含11列，不了解这个格式，内容如下：

```sh
lcl|Csinensis_orange1.1g033490m 109     466     txCdsPredict    .       507.5   1       1       1       109,    357,
lcl|Csinensis_orange1.1g033508m 109     466     txCdsPredict    .       507.5   1       1       1       109,    357,
lcl|Csinensis_orange1.1g033990m 39      360     txCdsPredict    .       521     1       1       1       39,     321,
lcl|Csinensis_orange1.1g033802m 682     1018    txCdsPredict    .       297.5   1       1       1       682,    336,

```

### step5. feature_extraction.pl 特征提取

输入：`cd-hit-est0.8.fasta` ` cd-hit-est0.8.cds`

输出：`cd-hit-est0.8.arff`  `cd-hit-est0.8.fasta.index`

```sh
$ /home/data/vip13t28/biosoft/RNAplonc/RNAplonc/feature_extraction.pl cd-hit-est0.8.fasta cd-hit-est0.8.cds > cd-hit-est0.8.arff
$ ls
cd-hit-est0.8.ar   cd-hit-est0.8.fasta        cd-hit-est0.8.fasta.index  test_.fasta
cd-hit-est0.8.cds  cd-hit-est0.8.fasta.clstr  test.fa
```

### step6. RNAplonc.model

Predictions on test data

输入：`RNAplonc.model`   `cd-hit-est0.8.arff`

RNAplonc.model 是安装包内的文件。

输出：`cd-hit-est0.8_end.txt`

获取帮助信息

```sh
java -cp /home/data/vip13t28/biosoft/RNAplonc/RNAplonc/weka.jar weka.classifiers.trees.REPTree 
```

运行

```sh
java -cp /home/data/vip13t28/biosoft/RNAplonc/RNAplonc/weka.jar weka.classifiers.trees.REPTree -l /home/data/vip13t28/biosoft/RNAplonc/RNAplonc/RNAplonc.model -T cd-hit-est0.8.arff  -p 0 >cd-hit-est0.8_end.txt
```

### step7 Filter result

结果过滤

输入：`cd-hit-est0.8.cds`  `cd-hit-est0.8_end.txt` 

输出：`cd-hit-est0.8_result.txt` 

```sh
/home/data/vip13t28/biosoft/RNAplonc/RNAplonc/FilterResults.py -c cd-hit-est0.8.cds -r cd-hit-est0.8_end.txt -o cd-hit-est0.8_result.txt -t 1 -p 0.5 
```

- `-t`  1为lncRNA，2为mRNA ，默认为全部输出。
- `-p`  `0到1质检的浮点数`。根据WEKA官方网站(https://waikato.github.io/weka-wiki/making_predictions/)，预测值是属于该类的概率。`>=0.5`将被视为`lncRNA`。用户可以`自由选择阈值`。然而，接近`1`时，您将获得`较少的假阳性`结果。

我没学过python，但还是看了下脚本，`FilterResults.py` 应该是没有设置默认值的。所以务必根据自己的认知自行设定阈值，我这里0.5只是用了手册中的示例。

## 帮助文档weka.jar  weka.classifiers.trees.REPTree

```sh
$ java -cp /home/data/vip13t28/biosoft/RNAplonc/RNAplonc/weka.jar weka.classifiers.trees.REPTree 

Weka exception: No training file and no object input file given.

General options:

-h or -help
	Output help information.
-synopsis or -info
	Output synopsis for classifier (use in conjunction  with -h)
-t <name of training file>
	Sets training file.
-T <name of test file>
	Sets test file. If missing, a cross-validation will be performed
	on the training data.
-c <class index>
	Sets index of class attribute (default: last).
-x <number of folds>
	Sets number of folds for cross-validation (default: 10).
-no-cv
	Do not perform any cross validation.
-force-batch-training
	Always train classifier in batch mode, never incrementally.
-split-percentage <percentage>
	Sets the percentage for the train/test set split, e.g., 66.
-preserve-order
	Preserves the order in the percentage split.
-s <random number seed>
	Sets random number seed for cross-validation or percentage split
	(default: 1).
-m <name of file with cost matrix>
	Sets file with cost matrix.
-toggle <comma-separated list of evaluation metric names>
	Comma separated list of metric names to toggle in the output.
	All metrics are output by default with the exception of 'Coverage' and 'Region size'.
	Available metrics:
	Correct,Incorrect,Kappa,Total cost,Average cost,KB relative,KB information,
	Correlation,Complexity 0,Complexity scheme,Complexity improvement,
	MAE,RMSE,RAE,RRSE,Coverage,Region size,TP rate,FP rate,Precision,Recall,
	F-measure,MCC,ROC area,PRC area
-l <name of input file>
	Sets model input file. In case the filename ends with '.xml',
	a PMML file is loaded or, if that fails, options are loaded
	from the XML file.
-d <name of output file>
	Sets model output file. In case the filename ends with '.xml',
	only the options are saved to the XML file, not the model.
-v
	Outputs no statistics for training data.
-o
	Outputs statistics only, not the classifier.
-do-not-output-per-class-statistics
	Do not output statistics for each class.
-k
	Outputs information-theoretic statistics.
-classifications "weka.classifiers.evaluation.output.prediction.AbstractOutput + options"
	Uses the specified class for generating the classification output.
	E.g.: weka.classifiers.evaluation.output.prediction.PlainText
-p range
	Outputs predictions for test instances (or the train instances if
	no test instances provided and -no-cv is used), along with the 
	attributes in the specified range (and nothing else). 
	Use '-p 0' if no attributes are desired.
	Deprecated: use "-classifications ..." instead.
-distribution
	Outputs the distribution instead of only the prediction
	in conjunction with the '-p' option (only nominal classes).
	Deprecated: use "-classifications ..." instead.
-r
	Only outputs cumulative margin distribution.
-z <class name>
	Only outputs the source representation of the classifier,
	giving it the supplied name.
-g
	Only outputs the graph representation of the classifier.
-xml filename | xml-string
	Retrieves the options from the XML-data instead of the command line.
-threshold-file <file>
	The file to save the threshold data to.
	The format is determined by the extensions, e.g., '.arff' for ARFF 
	format or '.csv' for CSV.
-threshold-label <label>
	The class label to determine the threshold data for
	(default is the first label)
-no-predictions
	Turns off the collection of predictions in order to conserve memory.

Options specific to weka.classifiers.trees.REPTree:

-M <minimum number of instances>
	Set minimum number of instances per leaf (default 2).
-V <minimum variance for split>
	Set minimum numeric class variance proportion
	of train variance for split (default 1e-3).
-N <number of folds>
	Number of folds for reduced error pruning (default 3).
-S <seed>
	Seed for random data shuffling (default 1).
-P
	No pruning.
-L
	Maximum tree depth (default -1, no maximum)
-I
	Initial class value count (default 0)
-R
	Spread initial count over all class values (i.e. don't use 1 per value)
-output-debug-info
	If set, classifier is run in debug mode and
	may output additional info to the console
-do-not-check-capabilities
	If set, classifier capabilities are not checked before classifier is built
	(use with caution).
-num-decimal-places
	The number of decimal places for the output of numbers in the model (default 2).
-batch-size
	The desired batch size for batch prediction  (default 100).
```

## 帮助文档FilterResults.py

```sh
$ FilterResults.py -h
usage: FilterResults.py [-h] -c CDSFILE -r RESULTFILE -o OUTPUTFILE [-t TYPE] [-p PERCENT]

optional arguments:
  -h, --help            show this help message and exit
  -c CDSFILE, --cdsFile CDSFILE
                        Path of the cds file, or the txCdsPredict output
  -r RESULTFILE, --resultFile RESULTFILE
                        Path of the result file, or the RNAplonc.model output
  -o OUTPUTFILE, --outputFile OUTPUTFILE
                        Path of the output file
  -t TYPE, --type TYPE  Filter the output type in the terminal, 1- lncRNA , 2-mRNA
  -p PERCENT, --percent PERCENT
                        Filter the output percentage in the terminal, float valeu between 0 and 1

```

