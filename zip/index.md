# Liunx常用解压缩命令

本文介绍了一些linux环境下常用的解压缩命令，linux环境下，每个压缩格式的打包和压缩的命令都不相同，咱又记不住每次都要问度娘，特此记录。
<!--more-->
### .tar 文件
~~~shell
tar -xvf FileName.tar         # 解包
tar -cvf FileName.tar DirName # 将DirName和其下所有文件（夹）打包
~~~
### .gz文件
~~~shell
gunzip FileName.gz  # 解压1
gzip -d FileName.gz # 解压2
gzip FileName       # 压缩，只能压缩文件
~~~
### .tar.gz文件、 .tgz文件
~~~shell
tar -zxvf FileName.tar.gz               # 解压
tar -zcvf FileName.tar.gz DirName       # 将DirName和其下所有文件（夹）压缩
tar -C DesDirName -zxvf FileName.tar.gz # 解压到目标路径
~~~
### .zip文件
~~~shell
unzip FileName.zip          # 解压
zip FileName.zip DirName    # 将DirName本身压缩
zip -r FileName.zip DirName # 压缩，递归处理，将指定目录下的所有文件和子目录一并压缩
~~~
### .rar文件
mac和linux并没有自带rar，需要去下载
~~~shell
rar x FileName.rar      # 解压  
rar a FileName.rar DirName # 压缩
~~~


