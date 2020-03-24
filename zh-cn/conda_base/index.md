# Conda_base

本文介绍了conda的基本操作命令，conda的api写的非常完整，有不明白的命令，直接在终端输入`conda 'command' --help`就能看到该命令的详细介绍
<!--more-->
## 常用的conda命令
* 查看环境列表
~~~shell
conda info --envs
~~~
* 创建新环境
~~~shell
conda create --name env-name python=3.5
~~~
* 激活新环境
~~~shell
conda activate env-name
~~~
* 关闭当前环境切换到base环境
~~~shell
conda activate
~~~
* 删除环境
~~~shell
conda remove --name env-name --all
~~~
* 搜索软件包
~~~shell
conda search opencv
# Name version build 
~~~
* 安装软件包
~~~shell
conda install name=version
conda update name=version
~~~
* 指定软件包来源
~~~shell
conda install -c spyder-ide spyder=3.0.0
#-c 表示从http://anaconda.org下载资源包
~~~
* 指定下载通道
~~~shell
conda install --channel https://conda.anaconda.org/menpo opencv3
~~~


