# Caffe(cpu) ubuntu安装

如果科研过程中需要用到前辈的caffe框架的源码，一定需要重新编译源码中的caffe框架，通常只要作者开放源码，其中一定会包含他们编译修改过的caffe文件。所以caffe框架的安装将是整个复现过程中最难的部分。
<!--more-->
caffe框架的学习难度为10级的话，那么9级一定是体现在caffe框架的安装上，我们先从简单的cpu版的caffe安装开始，不建议新手一开始就安装gpu版本，当然最好的教程一定是官方文档:[Caffe Tutorial](http://caffe.berkeleyvision.org/tutorial/)
## 安装依赖
~~~shell
sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler
sudo apt-get install --no-install-recommends libboost-all-dev
sudo apt-get install libatlas-base-dev
sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
~~~
## 配置makefile.config
修改caffe源码中的makefile.config文件，将以下代码的注释删掉就好
* 只使用CPU
~~~
CPU_ONLY := 1
~~~
* 如果想要使用python接口来写layer
~~~
WITH_PYTHON_LAYER := 1
~~~
* 配置python路径
~~~
ANACONDA_HOME  := $(HOME)/miniconda2
PYTHON_INCLUDE :=   $(ANACONDA_HOME)/include \
                    $(ANACONDA_HOME)/include/python2.7 \
                    $(ANACONDA_HOME)/lib/python2.7/site-packages/numpy/core/include 
~~~
* 设置hdf5的路径
~~~
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial 
~~~
## 编辑Makefile文件
很多教程都没有这一步，甚至是官方文档，也许这也是caffe较难安装的原因吧。
将：`LIBRARIES += glog gflags protobuf boost_system boost_filesystem m`
改为：`LIBRARIES += glog gflags protobuf boost_system boost_filesystem m hdf5_serial_hl hdf5_serial`
## make编译caffe
~~~shell
make all -j8  #j8可以同时使用8个线程编译，加快编译速度
make test
make runtest
~~~
## 编译python接口（使用python调用caffe框架）
~~~shell
make pycaffe
~~~

{{% admonition "warning" "注意" %}}
编译失败重新编译时，需要`make clean`
{{% /admonition %}}

## 导入caffe
~~~
import caffe 
~~~
若报错，找不到模块，需要把caffe下的Python路径导入环境变量中去
~~~shell
vim ~/.bashrc
~~~

{{% admonition "tip" "小贴士" %}}
vim编辑器用不习惯的话，可以使用`gedit`编辑器
{{% /admonition %}}

最后一行加上
~~~
export PYTHONPATH="/home/yaoting/caffe/python:$PYTHONPATH"
~~~
这里的路径写上你自己的路径
然后更新一下`bashrc`
~~~shell
source ~/.bashrc
~~~
## MNIST数据集测试
配置caffe完成后，我们可以利用MNIST数据集对caffe进行测试，过程如下：
1. 将终端定位到Caffe根目录（一定要在caffa的根目录下执行以下命令，否则会出现有些文件找不到的情况）
~~~
cd ~/caffe
~~~
2. 下载MNIST数据库并解压缩
~~~
./data/mnist/get_mnist.sh
~~~
3. 将其转换成Lmdb数据库格式
~~~
./examples/mnist/create_mnist.sh
~~~
4. 训练网络
~~~
./examples/mnist/train_lenet.sh
~~~
## 结尾
cpu版本的版本的caffe安装相对简单，建议操作完本文的安装方式以后再尝试安装gpu版本的caffe
