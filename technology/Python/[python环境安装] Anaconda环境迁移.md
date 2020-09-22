# Anaconda环境迁移

## 1. Anaconda 环境clone

```shell
mkdir /etc/conda
cd /etc/conda/

# 安装miniconda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh 
# source 环境变量
source  /home/wpzhou/.bashrc 
# 默认不启动
conda config --set auto_activate_base false

# 在其他已安装好环境的机器上打包环境，scp 到要安装的机器解压
tar -zcf python3.tar.gz /home/baina/miniconda3/envs/python3
scp python3.tar.gz  ip:~/tmp/
cd ~/tmp/
tar -zxf python3.tar.gz

# 克隆之前在其他机器安装好的python3环境
conda create --name python3 --clone /home/wpzhou/tmp/home/baina/miniconda3/envs/python3

# 将安装的新环境配置为python3，运行程序:python3 xx.py
rm /usr/bin/python3
ln -s /etc/conda/miniconda3/envs/python3/bin/python /usr/bin/python3
```

## 2. Anaconda 环境导入导出
```shell
#环境导出
#源机器导出环境
conda activate service3
conda env export > environment.yml
#环境导入
#将environment.yml文件拷贝到目标机器，在目标机器上导入
conda env create -f environment.yml
#编辑environment.yml文件，可以修改环境名称、安装目录、需要安装的包等

```