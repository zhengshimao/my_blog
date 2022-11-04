---
title:  我的conda常用命令、报错解决与技巧记录
tags:
  - conda
  - linux
keywords: “conda,linux”
categories:
 - 归档

date: 2022-11-04
  
toc: true
draft: false
url: post/conda_usage.html
---
conda软件是生信软件部署重要工具。大部分的生信工具可以通过conda安装，熟练使用conda也是生信学习的必备技能。

本文旨在记录我常用的命令，肯定记录的不全，而且有的内容介绍对于新手不是很友好。

## conda安装

下载地址

官网下载地址：https://www.anaconda.com/products/individual

北外镜像下载地址：https://mirrors.bfsu.edu.cn/anaconda/archive/

- linux版本安装

```sh
wget https://repo.anaconda.com/archive/Anaconda3-2021.11-Linux-x86_64.sh
bash Anaconda3-2021.11-Linux-x86_64.sh
#安装路径也选择默认，两次询问都选择yes。中间的查看信息可按q跳过。
```

最后一次询问是选择是否执行`conda init`  ，默认（也就是不输入yes直接回车）是不执行。如果选择不执行，那么后面还要手动添加conda环境变量

```sh
echo "export PATH=~/anaconda3/bin:$PATH" >> ~/.bashrc
source ~/.bashrc
```

如果选择执行（yes），会将在`.bashrc`添加类似如下。

```sh
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/your/home/anaconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/your/home/anaconda3/etc/profile.d/conda.sh" ]; then
        . "/your/home/anaconda3/etc/profile.d/conda.sh"
    else
        export PATH="/your/home/anaconda3/bin:$PATH" #添加环境变量
    fi
fi
unset __conda_setup
# <<< conda initialize <<<

```

上面内容我基本看不懂，但是体会到的用途：conda添加到环境变量；每次登录服务器会自动进入base环境。

## conda 配置

conda配置文件`~/.condarc` 。conda配置文件主要是添加源（或通道）。

常用通道`bioconda` 和 `conda-forge`

### 添加国内conda源

#### conda源介绍与添加

- 清华源

> https://mirror.tuna.tsinghua.edu.cn/help/anaconda/

添加源(我不太习惯官网的写法，还是按自己的添加)

```sh
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/simpleitk
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch-lts
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/menpo
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2s
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda
conda clean -i
```

- 北外源

> https://mirrors.bfsu.edu.cn/help/anaconda/

```sh
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/simpleitk
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/pytorch-lts
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/pytorch
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/menpo
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/msys2s
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/conda-forge
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/bioconda
conda clean -i
```

- 其他源

另外还有中科大源。其中*为你需要添加的conda源

> conda config --add channels  https://mirrors.bfsu.edu.cn/anaconda/cloud/*

#### conda源添加技巧

1）conda通道优先级

conda通道具有优先级，最后添加的通道或者位于`.condarc` 文件最上方通道优先级是最高的。这里说的“最后添加的通道”是指通过`conda config --add channels ` 命令添加通道。

2）添加官网给出的示例源之外的源

我们从`https://mirror.tuna.tsinghua.edu.cn/help/anaconda/` 看到的源只是清华源的一部分，这里我称之为“示例源”。

有点观察力和好奇心的人，可能会注意到conda源的写法，很相似，唯一不同的是最后的部分。那么，以清华源为例，共同的部分`http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/` 复制到浏览器地址栏是什么界面呢？如果你还没看过，现在就看一眼。该页面下的文件夹名称即为通道名称，如果你要添加这里面的源，例如 `qiime2` ，就可以在共同地址部分后面添加`qiime2` 即为`qiime2` 的清华源通道（全写：`https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/qiime2/`）。或者右击`https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/ ` 下的`qiime2` 文件夹，复制链接地址，也可得到了清华源的`qiime2`通道，然后将其添加进`.condarc` 即可。

注意：conda源并不是越多越好，过多的源会减慢你的软件安装速度。

#### 其它修改

1)添加其它非必需内容

```sh
conda config --set show_channel_urls yes
conda config --set auto_activate_base true
```

2)有时正常添加conda源后，会有网络原因的一些报错，可以打开`.condarc` 尝试按以下方法解决：

(1）删除默认通道

执行完添加通道命令后，默认通道` - defaults` 会自动添加到配置文件，尝试删除该行即可。

(2) 修改协议

将`https` 改为`http` 。对于网络协议，我不懂，有无隐患也暂时不知。

如果你始终因网络问题无法正常安装，还要考虑可能是网络问题。如果记不住`ip addr show` 或者`ifconfig` ，可以尝试`ping www.baidu.com` ，网络正常的话类似如下显示。其中`IP` 代替了具体的ip地址

```sh
PING www.a.shifen.com (IP) 56(84) bytes of data.
64 bytes from IP (IP): icmp_seq=1 ttl=128 time=36.6 ms
64 bytes from IP(IP): icmp_seq=2 ttl=128 time=36.4 ms
64 bytes from IP (IP): icmp_seq=3 ttl=128 time=36.4 ms
^C #这里是我执行了ctrl+c
--- www.a.shifen.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 36.448/36.544/36.696/0.245 ms

```

### 查看已配置的conda源

以下命令选其一即可。但是一般我选择最后一种，前几种命令太长，要么一时想不起来，要么懒得敲命令。

```sh
conda config --get channels #此命令会明确标出通道优先级，与你.condarc文件上下顺序相反
conda config --show channels
conda config --show-sources
cat ~/.condarc
```

### 删除源

```sh
conda config --remove channels
conda config --remove-key channels #换回默认源
```

这个操作我一般是不用的，即使源出了问题(清华源堵塞崩溃我已见怪不怪了)，我会直接删除`.condarc` 文件重新配置。

如果只是删除某个具体的通道，可以vim打开`.condarc` 文件直接删除。

### 我的`.condarc` 文件内容(清华源)

```sh
channels:
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/menpo/
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
show_channel_urls: true
auto_activate_base: true
```

清华源经有时会嘎掉，备下北外源

### 我的`.condarc` 文件内容（北外源）

```sh
channels:
  - http://mirrors.bfsu.edu.cn/anaconda/cloud/bioconda/
  - http://mirrors.bfsu.edu.cn/anaconda/cloud/conda-forge/
  - http://mirrors.bfsu.edu.cn/anaconda/pkgs/main/
  - http://mirrors.bfsu.edu.cn/anaconda/cloud/pytorch/
  - http://mirrors.bfsu.edu.cn/anaconda/cloud/menpo/
  - http://mirrors.bfsu.edu.cn/anaconda/cloud/msys2/
  - http://mirrors.bfsu.edu.cn/anaconda/pkgs/free/
show_channel_urls: true
auto_activate_base: true
```



## 环境激活与退出

对于我来说，激活与退出环境，命令过长，靠手指肌肉记忆很容易出错，我一般是使用别名。将下面内容放入到`.bashrc` 并`source ~/.bashrc` 

```sh
alias condact='conda activate'
alias deconda='conda deactivate'
```

激活环境，选其一即可

```sh
conda activate  #进入base环境
conda activate env_name #进入env_name环境
condact #进入base环境
condact env_name #进入env_name环境
source activate env_name
source ~/your_env/bin/activate #我conda-pack 进行环境迁移后激活your_env
```

退出环境，选其一即可

```sh
conda deactivate
deconda 
```

## conda环境创建与删除

环境创建

```sh
conda create -n env_name 
```

该环境位置`~/anaconda3/envs/env_name/`

环境删除，选其一即可

```sh
conda remove -n env_name  --all
conda env remove --name env_name
```

查看当前conda环境，选其一即可

```sh
conda env list
conda info -e
conda info --envs
```

环境克隆之类的命令，我暂时没用过，目前也觉得没有太大必要。有需要的请移步其它教程。

## 软件安装、更新、删除、搜索与查看

mamba是conda的提速器 ，手动安装或者conda安装均可。

> https://github.com/mamba-org/mamba

- 软件安装

```sh
conda/mamba install fastqc
conda/mamba install -y fastqc
conda/mamba install -y fastqc=0.11.9
conda/mamba install -c bioconda -c conda-forge -y fastqc=0.11.9
```

也可以在创建环境时安装软件

```sh
conda create -n env_name fastqc=0.11.9 -y
```

conda也可以本地安装一些软件，但是我用不到，也没用过，不做介绍，有需要自行查阅教程。

- 软件更新与删除

软件更新与删除我一般不使用，因为一些生信软件会指定某一些特定版本的依赖软件，随意更新可能会破坏依赖。另外conda clean 也可以完成软件删除操作，不过我没试过，如有需要自行查阅资料。

- 软件搜索

conda安装的软件名称，并不一定是官网给出的软件名称，对于不熟悉的软件会用到搜索命令。conda软件名称常常是：

软件名称全部为小写；

R包开头为`r-`

Perl模块开头为`perl` ，全部小写，中间的`::` 以`-` 代替。

常用的搜索命令

```sh
conda/mamba search fastqc
conda/mamba search fast* #搜索以fast开头的软件，结果不止是fastqc。
mamba repoquery search fastqc #据说这个命令更快，但是我没有体会到。
```

- 搜索的终极大招

搜索安装命令的**终极大招**`https://anaconda.org/` ，在`https://anaconda.org/` 界面下的搜索框中搜索即可。搜索perl模块时注意将`::` 改为`-` ，否则搜索不到。

- 软件查看

软件安装过多，可能不易查到需要的信息。可以使用管道符接`grep` 搜索。

```sh
conda list
conda list | grep ‘fastqc’
```

## 环境迁移

环境迁移，这里我只介绍conda-pack和yml文件两种方法，其它方法我没用过，看上去有些麻烦，请自行查阅资料学习。

### conda-pack完成环境迁移

这个方法我之前的“小白踩坑日记”记录过。

```sh
$ conda activate your_env_name #进入环境
$ conda install conda-pack -y #安装打包环境用的软件
$ conda pack -n your_env_name -o ~/file.tar.gz #打包环境为一个压缩文件。

#将压缩包file.tar.gz传输到新的服务器上。
zheng 21:50:57 ~  
$ mkdir test_env && cd test_env  #新建一个文件夹test_env，将file.tar.gz上传到该位置。

zheng 21:51:11 ~/test_env 
$ tar -zxvf file.tar.gz
$ source ./bin/activate  #激活环境 。不能用conda activate激活了。

(test_env) zheng 21:58:03 ~/test_env #注意观察，此时的环境名称为刚新建的文件夹test_env
$ conda-unpack  #注意观察，会有一个小小时间的等待。可以time conda-unpack看一下。
$ Trinity  #测试你打包的环境中软件能否调用
$ source deactivate  #conda deactivate 是无法退出的。
```

这里需要注意的是最后环境的激活方式，是用`source`完成的。

迁移后环境名称为刚新建的文件夹`test_env`

如果迁移后的环境，一段时间经常用到，可以将 `source ~/test_env/bin/activate` 放入自己的`.bashrc` 的最后，使其登录即可进入该环境。但是大佬说这么做有隐患，目前我还不知道具体有哪些隐患，大家慎重使用吧。大不了环境不常用之后，将其从`.bashrc`  删除即可。

## yml文件重现环境

- 导出环境内容

```sh
#导出环境内容
conda env export --file your_env.yml --name env_name
#your_env.yml文件中name:env_name名字为原环境名字，各软件版本都已明确指出。

#导出当前所处环境的yml文件
conda env export > your_env.yml 
```

- 重现环境

```sh
#创建一个新环境并安装其中软件
conda env create -f your_env.yml 
#后加-p 选项加参数可以指定路径安装。
```

新环境的名字为`your_env.yml` 中 第一行 `name:` 内容。

这种安装方式前面软件搜索步骤，可能依赖于服务器线程数和网速。因为只帮人安装过一次，没有过多测试，不是很确定。主要是想说，如果大家重现环境时比较慢，可能是正常现象，等就行了。

注意：这种迁移方式，只会迁移`conda install` 安装的软件。所以，软件安装时尽量全部使用conda。



