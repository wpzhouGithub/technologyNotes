# [python环境安装]  Anaconda更改安装路径

1. 将旧的安装目录mv到新的目录

   ```shell
   mv /home/sunny/miniconda3/ /root/miniconda3
   ```

2. 修改环境变量 ~/.bashrc， 将所有的/home/sunny/miniconda3/替换成/root/miniconda3； 更新.bashrc `source ~/.bashrc`

3. 修改conda的路径  vi /root/miniconda3/bin/conda  ， 将所有的/home/sunny/miniconda3/替换成/root/miniconda3

4. 更新conda配置: 如果不更新，conda命令虽然可以运行，但是会找不到.condarc文件，还是会有问题，更新方式为：`conda update conda`