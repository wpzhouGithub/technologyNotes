@[TOC](Ubuntu **Anaconda**安装)

## **一、什么是Anaconda？**

**1. 简介**

Anaconda（[官方网站](https://link.zhihu.com/?target=https%3A//www.anaconda.com/download/%23macos)）就是可以便捷获取包且对包能够进行管理，同时对环境可以统一管理的发行版本。Anaconda包含了conda、Python在内的超过180个科学包及其依赖项。



**2. 特点**

Anaconda具有如下特点：

▪ 开源

▪ 安装过程简单

▪ 高性能使用Python和R语言

▪ 免费的社区支持

其特点的实现主要基于Anaconda拥有的：

▪ conda包

▪ 环境管理器

▪ 1,000+开源库

如果日常工作或学习并不必要使用1,000多个库，那么可以考虑安装Miniconda（[下载界面请戳](https://docs.conda.io/projects/continuumio-conda/en/latest/user-guide/install/linux.html)），这里不过多介绍Miniconda的安装及使用。



**3. Anaconda、conda、pip、virtualenv的区别**

**① Anaconda**

Anaconda是一个包含180+的科学包及其依赖项的发行版本。其包含的科学包包括：conda, numpy, scipy, ipython notebook等。



**② conda**

conda是包及其依赖项和环境的管理工具。

▪ 适用语言：Python, R, Ruby, Lua, Scala, Java, JavaScript, C/C++, FORTRAN。

▪ 适用平台：Windows, macOS, Linux

▪ 用途：

① 快速安装、运行和升级包及其依赖项。

② 在计算机中便捷地创建、保存、加载和切换环境。

> 如果你需要的包要求不同版本的Python，你无需切换到不同的环境，因为conda同样是一个环境管理器。仅需要几条命令，你可以创建一个完全独立的环境来运行不同的Python版本，同时继续在你常规的环境中使用你常用的Python版本。——[Conda官方网站](https://link.zhihu.com/?target=https%3A//conda.io/docs/)

▪ conda为Python项目而创造，但可适用于上述的多种语言。

▪ conda包和环境管理器包含于Anaconda的所有版本当中。



**③ pip**

pip是用于安装和管理软件包的包管理器。

▪ pip编写语言：Python。

▪ Python中默认安装的版本：

① Python 2.7.9及后续版本：默认安装，命令为 **pip**

② Python 3.4及后续版本：默认安装，命令为 **pip3**

▪ pip名称的由来：pip采用的是**递归缩写**进行命名的。其名字被普遍认为来源于2处：

① “Pip installs Packages”（“pip安装包”）

② “Pip installs Python”（“pip安装Python”）



**④ virtualenv**

virtualenv是用于创建一个**独立的**Python环境的工具。

▪ 解决问题：

1. 当一个程序需要使用Python 2.7版本，而另一个程序需要使用Python 3.6版本，如何同时使用这两个程序？如果将所有程序都安装在系统下的默认路径，如：**/usr/lib/python2.7/site-packages**，当不小心升级了本不该升级的程序时，将会对其他的程序造成影响。
2. 如果想要安装程序并在程序运行时对其库或库的版本进行修改，都会导致程序的中断。
3. 在共享主机时，无法在全局 **site-packages** 目录中安装包。

▪ virtualenv将会为它自己的安装目录创建一个环境，这并**不与**其他virtualenv环境共享库；同时也可以**选择性**地不连接已安装的全局库。



**⑤ pip 与 conda 比较**

**→ 依赖项检查**

▪ pip：

① **不一定**会展示所需其他依赖包。

② 安装包时**或许**会直接忽略依赖项而安装，仅在结果中提示错误。

▪ conda：

① 列出所需其他依赖包。

② 安装包时自动安装其依赖项。

③ 可以便捷地在包的不同版本中自由切换。



**→ 环境管理**

▪ pip：维护多个环境难度较大。

▪ conda：比较方便地在不同环境之间进行切换，环境管理较为简单。



**→ 对系统自带Python的影响**

▪ pip：在系统自带Python中包的更新/回退版本/卸载将影响其他程序。

▪ conda：不会影响系统自带Python。



**→ 适用语言**

▪ pip：仅适用于Python。

▪ conda：适用于Python, R, Ruby, Lua, Scala, Java, JavaScript, C/C++, FORTRAN。



**⑥ conda与pip、virtualenv的关系**

▪ conda**结合**了pip和virtualenv的功能。



## **二、Anaconda的适用平台及安装条件**

**1. 适用平台**

Anaconda可以在以下系统平台中安装和使用：

▪ Windows

▪ macOS

▪ Linux（x86 / Power8）



**2. 安装条件**

▪ 系统要求：32位或64位系统均可

▪ 下载文件大小：约500MB

▪ 所需空间大小：3GB空间大小（Miniconda仅需400MB空间即可）



## 三、Anaconda 安装及使用

如前文所说，如果不需要Anaconda里的众多包，可以安装Miniconda

#### 1. 下载安装

   ```shell
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
# 根据提示yes/no
bash Miniconda3-latest-Linux-x86_64.sh  
   ```

#### 2. 查看conda 版本

```conda --version```

#### 3. 更新conda

```conda update conda```

#### 4. 查看conda帮助
```shell
conda --help
#或
conda -h
```

#### 5. 新建虚拟环境

```conda create --name <env_name> <package_names>```
注意：

- <env_name>即创建的环境名。建议以英文命名，且不加空格，名称两边不加尖括号“<>”

- <package_names>即安装在环境中的包名。名称两边不加尖括号“<>”。如果要安装指定的版本号，则只需要在包名后面以=和版本号的形式执行。

  如：conda create --name python2 python=2.7，即创建一个名为“python2”的环境，环境中安装版本为2.7的python。

- 如果要在新创建的环境中创建多个包，则直接在<package_names>后以空格隔开，添加多个包名即可。

  如：conda create -n conda-test python=3.6 numpy pandas，即创建一个名为“conda-test ”的环境，环境中安装版本为3.6的python，同时也安装了numpy和pandas。

- –name同样可以替换为-n。


#### 6. 切换conda环境

```conda activate env_name```

#### 7. 退出虚拟环境

```conda deactivate```

#### 8. 显示安装过的所有虚拟环境
```shell
conda info --envs
#或
conda info -e
#或
conda env list
```
#### 9. 复制环境

```conda create --name new_env_name --clone copied_env_name```
注意：

- copied_env_name即为被复制/克隆环境名。

- new_env_name即为复制之后新环境的名称。

#### 10. 删除环境

```conda remove --name env_name --all```

#### 11. 包管理

```shell
#精确查找包
conda search --full-name 查找的包名
#模糊查找包
conda search 查找的内容
#获取当前环境中已安装的包信息
conda list
#指定环境安装包
conda install --name 环境名 要安装的包名
#要指定版本 在conda-test环境中安装 django并制定版本为2.0.6
conda install --name conda-test django=2.0.6
#在当前环境中安装包
conda install 要安装的包名
```
#### 12. 如果conda安装不上的包，或者没有的包，可以使用pip安装
```pip install 安装的包名```

#### 13. 跨机器克隆conda环境

    ```
    #将原机器的目标目录copy到新机器的目录，然后调研clone命令
    conda create -n BBB --clone ~/path
    #但是 移植过来的环境只是安装了你原来环境里用conda install等命令直接安装的包，你用pip之类装的东西没有移植过来，需要你重新安装
    ```


​    

