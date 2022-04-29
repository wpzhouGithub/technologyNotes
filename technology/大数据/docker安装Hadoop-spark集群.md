# 使用docker安装Hadoop、Spark集群

## 项目介绍

- 使用docker安装Hadoop、Spark的测试集群
- 主要思路：
  1. 一台container里面构建单例的环境
  2. 保存为image
  3. 基于image启动多台container作为集群

## 环境介绍

- docker：Windows10+docker 20.10.11
- Hadoop：3.3.1
- Spark：3.2.0
- JDK：1.8
- Scala：2.12
- Linux：Ubuntu20

## 资源下载

```bash
#hadoop
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.1/hadoop-3.3.1.tar.gz

#spark
wget https://www.apache.org/dyn/closer.lua/spark/spark-3.2.0/spark-3.2.0-bin-hadoop3.2.tgz

```



## docker安装

- Windows docker安装

  - 参考https://docs.docker.com/desktop/windows/install/

- 安装Ubuntu

  ```bash
   # 获取镜像
   docker pull ubuntu:20.04
   # 运行容器
   docker run -it ubuntu /bin/bash
   # 启动容器
   docker start 03cbf76b09a4
   # 进入容器
   docker attach 03cbf76b09a4
  
  ```

- 更新安装源(**源一定要跟Ubuntu版本一致**)

  ```bash
  mv /etc/apt/sources.list /etc/apt/sourses.list.backup
  
  echo "deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse" > /etc/apt/sources.list
  ```

  

## 环境安装

### Java环境

- 安装Java

```bash
apt-cache search jdk
apt-get install openjdk-8-jdk
java -version
```

- 配置jdk

  - **设置java 环境变量,配置profile**

    ```bash
    vi /etc/profile
    ```

  - 在文件末尾加上

    ```
    JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64/
    JAVA_BIN=/usr/lib/jvm/java-1.8.0-openjdk-amd64/bin
    JRE_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64/jre
    CLASSPATH=/usr/lib/jvm/java-1.8.0-openjdk-amd64/jre/lib:/usr/lib/jvm/java-1.8.0-openjdk-amd64/lib:/usr/lib/jvm/java-1.8.0-openjdk-amd64/jre/lib/charsets.jar
    export  JAVA_HOME  JAVA_BIN JRE_HOME  PATH  CLASSPATH
    ```

  - 使得配置立马有效

  ```
  source /etc/profile
  ```

  - 配置 bashrc

  ```
  vi ~/.bashrc
  ```

  - 在文件末尾加上

  ```
  export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
  export PATH=$JAVA_HOME/bin:$PATH
  export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
  ```

  使得配置立马有效

  ```
  source ~/.bashrc
  ```

  查看成功:`java -version`

### Scala环境

- 配置环境变量

```
vim /etc/profile
export SCALA_HOME=/opt/scala-2.12.12
export PATH=$PATH:$SCALA_HOME/bin
```

  - 环境变量生效

```
source /etc/profile
```

  - 查看scala版本 

```
scala -version
```

### python环境

```bash
wget https://www.python.org/ftp/python/3.9.0/Python-3.9.0.tgz
gzip -d Python-3.9.0.tgz
tar -xf Python-3.9.0.tar
cd Python-3.9.0
# 参考README.rst 手动便宜安装

./configure
make
make test
sudo make install
```



### ssh 安装

```bash
apt install net-tools
apt install openssh-server
apt install openssh-client
cd ~
ssh-keygen -t rsa -P ""

cat .ssh/id_rsa.pub >> .ssh/authorized_keys
service ssh start
ssh 127.0.0.1
```

 　为了保证系统启动的时候也启动ssh服务，我们将启动命令 “service ssh start” 放到bashrc文件末尾中。

```
vim ~/.bashrc
```

　

## Hadoop安装

- 环境配置

  ```bash
  mv hadoop-3.3.1 /opt/
  ```

  ```
 vi /opt/hadoop-3.3.1/etc/hadoop/hadoop-env.sh
  ```
  
  在文件末尾添加jdk目录（这里=后面添加的是你的jdk目录）

  ```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64/
  
  ```
  
  检查安装的hadoop是否可用 `./bin/hadoop version` （注意要在hadoop的安装目录下执行）

  

  前面在验证hadoop命令的时候需要在hadoop的安装目录下执行./bin/hadoop，为了方便在任意地方执行hadoop命令，配置hadoop的全局环境变量，与java一样，修改~/.bashrc文件

  执行`vi ~/.bashrc`

  添加内容

  ```
export HADOOP_HOME=/opt/hadoop-3.3.1
  export HADOOP_CONF_DIR=/opt/hadoop-3.3.1/etc/hadoop
  export PATH=$PATH:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin
  ```
  
  `hadoop version` 验证变量生效

  ```bash
source .bashrc
  hadoop version
  ```
  
- 机器分配

  - 集群共有4台机器，分布为hadoop0-1、hadoop0-2、hadoop1-1、hadoop1-2；其中hadoop0-1为NameNode、hadoop0-2为ResourceManager

- Hadoop配置文件配置

  - etc/hadoop/core-site.xml

    ```bash
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://hadoop0-1:9000</value>
        </property>
        <property>
            <name>hadoop.tmp.dir</name>
            <value>/root/hadoop/tmp</value>
        </property>
    </configuration>
    ```

    

  - etc/hadoop/hdfs-site.xml

    ```bash
    <configuration>
        <property>
            <name>dfs.namenode.name.dir</name>
            <value>/root/hadoop/hdfs/name</value>
        </property>
        
        <property>
            <name>dfs.replication</name>
            <value>2</value>
        </property>
    
        <property>
            <name>dfs.namenode.data.dir</name>
            <value>/root/hadoop/hdfs/data</value>
        </property>
    </configuration>
    ```

    

  - etc/hadoop/yarn-site.xml

    ```bash
    <configuration>
        <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>hadoop0-2</value>
        </property>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
    </configuration>
    ```

    

  - etc/hadoop/mapred-site.xml

    ```bash
    <configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
        
        
              <property>
                      <name>yarn.app.mapreduce.am.env</name>
                      <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
                </property>
              <property>
                          <name>mapreduce.map.env</name>
                            <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
                    </property>
                    <property>
                            <name>mapreduce.reduce.env</name>
                              <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
                      </property>
                        
                        
    </configuration>
    ```
  
    
  
  - etc/hadoop/workers
  
    ```bash
    hadoop1-1
    hadoop1-2
    ```
  
  - 格式化hdfs
  
    ```bash
    hdfs namenode -format
    ```
  
    

## Spark安装

```bash
mv spark-3.2.0-bin-hadoop3.2 /opt/spark-3.2.0-bin-hadoop3.2/

# vim /etc/profile
export SPARK_HOME=/opt/spark-3.2.0-bin-hadoop3.2/
export PATH=$PATH:$SPARK_HOME/bin

```



## 集群安装

```bash
# commit之前先格式化HDFS
hdfs namenode -format

# docker 生成镜像
docker commit -m "hadoop3.3 spark3.2 ubuntu20 install" 60c06e4df2f6 ubuntu:hadoop

# 我们创建一个hadoop机器环境内所需要的网络环境。同一个网络环境下可以直接进行访问
docker network create --driver=bridge hadoop

# 查看创建了哪些网络环境
docker network ls

# hadoop集群机器
docker run -itd --network hadoop -h hadoop0-1 --name hadoop0-1 -p 9870:9870 ubuntu:hadoop 
docker run -itd --network hadoop -h hadoop0-2 --name hadoop0-2 -p 8088:8088 ubuntu:hadoop 
docker run -itd --network hadoop -h hadoop1-1 --name hadoop1-1 ubuntu:hadoop 
docker run -itd --network hadoop -h hadoop1-2 --name hadoop1-2 ubuntu:hadoop 

# 到resourcemanager机器（hadoop0-2）上启动机器
cd /opt/hadoop-3.3.1/sbin
./start-all.sh

# HDFS 创建测试目录
hadoop fs -mkdir -p /data/test/input/ /data/test/output/

# mapred job 测试
hadoop jar /opt/hadoop-3.3.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.1.jar wordcount /data/test/input/input.txt /data/test/output/mr-example/

# spark 测试
cd /opt/spark-3.2.0-bin-hadoop3.2
spark-submit --class org.apache.spark.examples.SparkPi \
    --master yarn \
    --deploy-mode cluster \
    --driver-memory 4g \
    --executor-memory 2g \
    --executor-cores 1 \
    /opt/spark-3.2.0-bin-hadoop3.2/examples/jars/spark-examples*.jar 2
    
# 将spark需要的jar文件上传至hdfs，避免每次运行作业都上传
hadoop fs -put /opt/spark-3.2.0-bin-hadoop3.2/jars/*  /user/spark/spark_jars/
# vi /opt/spark-3.2.0-bin-hadoop3.2/conf/spark-defaults.conf
spark.yarn.jars=hdfs://hadoop0-1:9000//user/spark/spark_jars/*

```

