---
type: blog
title: 虚拟机 Hadoop 环境配置
date: 2020-09-17
categories: 教程
tags: Hadoop, 教程
cover: false
meta:
  date: false
---

## 一、安装 CentOS 6 & 基本配置

> win10通过VMware安装CentOS6.5 - 简书
> https://www.jianshu.com/p/9d5b9757a1ef

1、关闭防火墙

```bash
service iptables stop
chkconfig iptables off
```

2、创建普通用户

```bash
useradd Ace
passwd 123
```

3、创建软件存储文件夹，并更改所有权

```bash
mkdir /opt/software /opt/module
chown Ace:Ace /opt/software /opt/module
```

4、用户添加到 sudoers

```bash
vi /etc/sudoers
Ace ALL=(ALL) NOPASSWD:ALL
```

5、改 Hosts

```bash
#!/bin/bash
for ((i=101;i<105;i++))
do
        echo "192.168.87.$i hadoop$i" >> /etc/hosts
done
```

6、改静态 ip

`vim /etc/sysconfig/network-scripts/ifcfg-eth0`

```bash
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.87.100
PREFIX=24
GATEWAY=192.168.87.2
DNS1=192.168.87.2
NAME="System eth0"
```

【创建新虚拟机，下面的都要做一遍，可以写脚本解决（看下面，推荐）】

6、改 ip 地址（同上6）

`vim /etc/sysconfig/network-scripts/ifcfg-eth0`

```bash
IPADDR=192.168.87.100   # 改成对应的
```

7、改主机名

`vim /etc/sysconfig/network`

```bash
HOSTNAME=hadoopxxx
```

8、删除多余网卡

`vim /etc/udev/rules.d/70-persistent-net.rules`

```bash
# 第一行删掉（只保留一个网卡就行，注释也删掉，手动删的话记得和后面脚本的行数要对应上）
# 第二行最后 NAME="eth1" 改为 NAME="eth0"
```

9、拍快照，克隆

【脚本】

- 分发脚本 xsync

`vim xsync`

```bash
#!/bin/bash
# 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if ((pcount==0)); then
echo no args;
exit;
fi

p1=$1
fname=`basename $p1`
echo fname=$fname

dirname=`cd -P $(dirname $p1); pwd`
echo dirname=$dirname

user=`whoami`

for((i=102;i<105;i++)); do
    echo "------------- hadoop$i ------------"
    rsync -avlP $dirname/$fname hadoop$i:$dirname
done
```

移动到`bin`目录下，`sudo mv xsync /bin`

安装`rsync`，`sudo yum install -y rsync`

改权限，`chmod +x xsync`

- 执行相同命令脚本

```bash
#!/bin/bash
pcount=$#
if ((pcount==0)); then
echo no args;
exit;
fi

for i in hadoop102 hadoop103 hadoop104
do
        echo "==== $i $1 ===="
        ssh $i "$1"
done
```

移动到`/bin`，改权限

- 自动配置网络脚本

```bash
#!/bin/bash
id=$1
# sed -i "s/before replace/after replace/"
sudo sed -i "s/192.168.87.101/192.168.87.$id/" /etc/sysconfig/network-scripts/ifcfg-eth0
sudo sed -i "s/hadoop101/hadoop$id/" /etc/sysconfig/network

file=/etc/udev/rules.d/70-persistent-net.rules

# count "SUBSYSTEM" word number
nu=$(grep -c SUBSYSTEM $file)
if(($nu > 1));
then
        # delete 8th line
        sed -i '8d' $file
fi

sed -i 's/eth1/eth0/' $file
reboot
```

改权限，`chmod +x change_network`



## 二、安装 JAVA 和 Hadoop

1、下载/上传 java 和 hadoop 的包到 `/opt/software`

2、解压 java 和 hadoop 到 `/opt/module`

3、安装（配置环境变量）

```bash
sudo vim /etc/profile
# 在末尾添加

# JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin

# HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-2.7.2
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

4、测试安装情况

执行下面命令后，能出现版本号即为成功

```bash
$ java -version
$ hadoop version
```

5、使用 xsync 同步到多个机器上



## 三、Hadoop 配置

[Apache Hadoop 3.2.1 – Hadoop: Setting up a Single Node Cluster.](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)

- 安装插件

```bash
$ sudo yum install ssh
$ sudo yum install pdsh
  
# pdsh 可能默认找不到
$ wget http://mirrors.mit.edu/epel/6/i386/epel-release-6-8.noarch.rpm
$ rpm -Uvh epel-release-6-8.noarch.rpm
$ yum install pdsh
```

- 环境配置

`vim etc/hadoop/hadoop-env.sh`

```bash
export JAVA_HOME=/your-java-home-path
```

### 3.1 本地运行模式

```bash
$ mkdir input
$ cp etc/hadoop/*.xml input
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep input output 'dfs[a-z.]+'
$ cat output/*
```

### 3.2 伪分布式

1、配置

`vim etc/hadoop/core-site.xml`

```xml
<configuration>
  	<!-- 指定HDFS中NameNode的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
  	
  	<!-- 指定Hadoop运行时产生文件的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-2.7.2/data/tmp</value>
    </property>
</configuration>
```

`etc/hadoop/hdfs-site.xml`

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

**暂时不配置 yarn 了**

`etc/hadoop/yarn-env.sh`

```bash
export JAVA_HOME=/opt/module/jdk1.8.0_144
```

2、设置免密码登录

```bash
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
```

3、执行

1. Format the filesystem:

   ```
   $ bin/hdfs namenode -format
   ```

2. Start NameNode daemon and DataNode daemon:

   ```
   $ sbin/start-dfs.sh
   ```

   The hadoop daemon log output is written to the `$HADOOP_LOG_DIR` directory (defaults to `$HADOOP_HOME/logs`).

3. Browse the web interface for the NameNode; by default it is available at:

   - NameNode - `http://localhost:9870/`

4. Make the HDFS directories required to execute MapReduce jobs:

   ```
   $ bin/hdfs dfs -mkdir /user
   $ bin/hdfs dfs -mkdir /user/<username>
   ```

5. Copy the input files into the distributed filesystem:

   ```
   $ bin/hdfs dfs -mkdir input
   $ bin/hdfs dfs -put etc/hadoop/*.xml input
   ```

6. Run some of the examples provided:

   ```
   $ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar grep input output 'dfs[a-z.]+'
   ```

7. Examine the output files: Copy the output files from the distributed filesystem to the local filesystem and examine them:

   ```
   $ bin/hdfs dfs -get output output
   $ cat output/*
   ```

   or View the output files on the distributed filesystem:

   ```
   $ bin/hdfs dfs -cat output/*
   ```

8. When you’re done, stop the daemons with:

   ```
   $ sbin/stop-dfs.sh
   ```

### 3.3 完全分布式

同步两个软件，/etc/profile

#### 3.3.1 集群配置

- 集群部署规划

NN 1个； 2NN 1个；RM 1个；DN 3个、NM 3个 —— 最少共需六台机器，但是开不起那么多个虚拟机，因此按下表进行合并配置

|      | hadoop102          | hadoop103                    | hadoop104                   |
| ---- | ------------------ | ---------------------------- | --------------------------- |
| HDFS | NameNode, DataNode | DataNode                     | SecondaryNameNode, DataNode |
| YARN | NodeManager        | ResourceManager, NodeManager | NodeManager                 |

- 配置集群

**1）核心配置文件**

配置`etc/hadoop/core-site.xml`

```xml
<!-- 指定HDFS中NameNode的地址 -->
<property>
		<name>fs.defaultFS</name>
		<value>hdfs://hadoop102:9000</value>
</property>

<!-- 指定Hadoop运行时产生文件的存储目录 -->
<property>
		<name>hadoop.tmp.dir</name>
		<value>/opt/module/hadoop-2.7.2/data/tmp</value>
</property>
```

**2）HDFS配置文件**

配置``etc/hadoop/hadoop-env.sh`

```
export JAVA_HOME=/opt/module/jdk1.8.0_144
```

配置`etc/hadoop/hdfs-site.xml`

```xml
<property>
		<name>dfs.replication</name>
  	<value>3</value>
</property>

<!-- 指定Hadoop辅助名称节点主机配置 -->
<property>
   <name>dfs.namenode.secondary.http-address</name>
   <value>hadoop104:50090</value>
</property>
```

**3）YARN配置文件**

配置`etc/hadoop/yarn-env.sh`

```
export JAVA_HOME=/opt/module/jdk1.8.0_144
```

配置`etc/hadoop/yarn-site.xml`

```xml
<!-- Reducer获取数据的方式 -->
<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
</property>

<!-- 指定YARN的ResourceManager的地址 -->
<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>hadoop103</value>
</property>
```

**4）MapReduce配置文件**

配置`etc/hadoop/mapred-env.sh`

```
export JAVA_HOME=/opt/module/jdk1.8.0_144
```

配置`etc/hadoop/mapred-site.xml`

```xml
$ cp mapred-site.xml.template mapred-site.xml
~
~
<!-- 指定MR运行在Yarn上 -->
<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
</property>
~
~
```

- 在集群上分发配置好的Hadoop配置文件

```bash
$ xsync /opt/module/hadoop-2.7.2/
```

#### 3.3.2 集群单点启动

1）如果集群是第一次启动，需要格式化NameNode

```bash
$ hadoop namenode -format
```

2）在 hadoop102 上启动 NameNode

```bash
$ hadoop-daemon.sh start namenode
```

3）在 hadoop102、hadoop103、hadoop104 上**分别**启动DataNode

```bash
$ hadoop-daemon.sh start datanode
```

4）在 hadoop104 上启动 secondarynamenode

```bash
$ hadoop-daemon.sh start secondarynamenode
```

5）查看服务启动情况 jps

```
[hadoop102]$ jps
4182 Jps
2842 DataNode
2747 NameNode

[hadoop103]$ jps
1712 DataNode
2215 Jps

[hadoop104]$ jps
1680 DataNode
2266 Jps
2171 SecondaryNameNode
```

#### 3.3.3 集群一键启动

- 配置机器间 ssh 无密登录

「方法1：共用一个秘钥」

```bash
# 生成秘钥
$ ssh-keygen -t rsa
# 将秘钥添加到本机的 authorized_keys 中，实现本机无密登录
$ ssh-copy-id hadoop102
# 共享同一个秘钥，实现集群机器间无密登录
$ xsync ~/.ssh
```

「方法2：每个机器单独生成秘钥」

```bash
# 在每个机器上生成秘钥（每个机器执行一遍）
$ ssh-keygen -t rsa
# 把每个机器的秘钥分发到别的机器上（每个机器执行一遍）
$ ssh-copy-id hadoop102
$ ssh-copy-id hadoop103
$ ssh-copy-id hadoop104
```

- 添加机器名，配置`etc/hadoop/slaves`，并同步到其他机器（xsync）

```
hadoop102
hadoop103
hadoop104
```

- 启动

```bash
[hadoop102]$ start-dfs.sh
[hadoop103]$ start-yarn.sh  # 应该在ResouceManager所在的机器上启动YARN
```

- 测试（单词计数）

```bash
# 创建一个文件夹 wcinput，里面放一个文件，添加计数文件中的内容，如：
qwe
ddd
smo
123
qwe

# 将这个文件夹上传到 hdfs
$ hadoop fs -put wcinput/ /

# 执行 MapReduce 程序
$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /wcinput /output

# 查看执行结果
[Ace@hadoop102 hadoop-2.7.2]$ hadoop fs -ls /output
Found 2 items
-rw-r--r--   3 Ace supergroup          0 2020-09-27 10:15 /output/_SUCCESS
-rw-r--r--   3 Ace supergroup         24 2020-09-27 10:15 /output/part-r-00000
[Ace@hadoop102 hadoop-2.7.2]$ hadoop fs -cat /output/part-r-00000
123	1
ddd	1
qwe	2
smo	1
```

- 停止

```bash
[hadoop102]$ stop-dfs.sh
[hadoop103]$ stop-yarn.sh
```

#### 3.3.4 配置历史服务器 & 日志聚集

- 配置`etc/hadoop/mapred-site.xml`，添加：

```xml
    <!-- 历史服务器端地址 -->
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>hadoop104:10020</value>
    </property>
    <!-- 历史服务器web端地址 -->
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>hadoop104:19888</value>
    </property>
```

- 配置`etc/hadoop/yarn-site.xml`

```xml
    <!-- 日志聚集功能使能 -->
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
    <!-- 日志保留时间设置7天 -->
    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
    </property>
```

- [分发配置]

- 启动

```bash
[hadoop102]$ start-dfs.sh
[hadoop103]$ start-yarn.sh
[hadoop104]$ mr-jobhistory-daemon.sh start historyserver
$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /wcinput /output1
```

- 查看
  - 打开 yarn web http://hadoop103:8088/
  - 打开 history，再点 log 就能看到任务具体的日志信息了
  - ![image-20201008173041985](../../../../../Library/Application Support/typora-user-images/image-20201008173041985.png)

#### 3.3.5 集群时间同步

1）检查 ntp 是否安装

查看 ntp 包（切换到 root 用户）

```bash
[root@hadoop102 ~]# rpm -qa | grep ntp
ntp-4.2.6p5-15.el6.centos.x86_64
ntpdate-4.2.6p5-15.el6.centos.x86_64
```

如果没有这两个服务要安装一下

```bash
yum install -y ntp
```

先停止 ntp 服务

```bash
# 查看 ntpd 服务是否在运行
$ service ntpd status
# 如果在运行先关闭
$ service ntpd stop
$ chkconfig ntpd off
# 再查看一下 ntpd 运行情况
$ chkconfig --list ntpd
```

2）修改ntp配置文件`/etc/ntp.conf`

```bash
# 修改1（授权192.168.1.0-192.168.1.255网段上的所有机器可以从这台机器上查询和同步时间）
restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap

# 修改2（集群在局域网中，不使用其他互联网上的时间），将下面四行注释掉
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst

# 添加3（当该节点丢失网络连接，依然可以采用本地时间作为时间服务器为集群中的其他节点提供时间同步）
server 127.127.1.0
fudge 127.127.1.0 stratum 10
```

3）修改`/etc/sysconfig/ntpd`文件

```bash
增加内容如下（让硬件时间与系统时间一起同步）
SYNC_HWCLOCK=yes
```

4）重新启动ntpd服务

```bash
$ service ntpd status
$ service ntpd start
```

5）设置ntpd服务开机启动

```bash
[hadoop102]$ chkconfig ntpd on
```

6）其他机器配置（必须root用户）

在其他机器配置10分钟与时间服务器同步一次

```bash
$ crontab -e
*/10 * * * * /usr/sbin/ntpdate hadoop102
```

测试（修改任意机器时间），十分钟后查看机器是否与时间服务器同步

```bash
$ date -s "2017-9-11 11:11:11"
```

7）若主机时间不对

```bash
$ service ntpd stop
$ ntpdate us.pool.ntp.org
$ service ntpd start

# 或者直接
$ sudo ntpdate -u pool.ntp.org
```

### 3.4 常用端口记录

> [Hadoop默认端口应用一览_在路上的学习者-CSDN博客](https://blog.csdn.net/yeruby/article/details/49406073)
>
> [Hadoop常用端口号_baiBenny的博客-CSDN博客](https://blog.csdn.net/baiBenny/article/details/53887328)

| 组件 | 节点     | 默认端口 | 配置                      | 用途说明                             |
| ---- | -------- | -------- | ------------------------- | ------------------------------------ |
| HDFS | NameNode | 50070    | dfs.namenode.http-address | http服务的端口，可查看 HDFS 存储内容 |
| HBase | Master | 16000 |  | Master RPC Port（远程通信调用） |
|  | Master | 16010 |  | Master Web Port |
|  | Regionserver | 16020 |  | Regionserver RPC Port |
|  | Regionserver | 16030 |  | Regionserver Web Port |
| Spark |  | 4040 |  | 查看 Spark Job |

### 3.5 HA

- 配置两个 NameNode
- 配置 JournalNode 用于将 Active NN 的数据 同步到 Standby NN 上「解决元数据同步的问题」
- 配置 Zookeeper，解决主备 NN 切换的问题，防止脑裂
- 在 NN 上启动 failoverController（zkfc），作为 Zookeeper 的客户端，实现与 zk 集群的交互和监测

<img src="https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/image-20201110104755190.png" alt="image-20201110104755190" style="zoom: 33%;" />

## 四、Zookeeper 配置

### 4.1 本地模式

**1、安装前准备**

- 安装Jdk
- 拷贝Zookeeper安装包到Linux系统下
- 解压到指定目录

```bash
tar -zxvf zookeeper-3.4.10.tar.gz -C /opt/module/
```

**2、配置修改**

- 修改`conf/zoo_sample.cfg`

```bash
cp zoo_sample.cfg zoo.cfg
```

- 修改 zoo.cfg 文件

```bash
dataDir=/opt/module/zookeeper-3.4.10/zkData
```

并创建 zkData 文件夹

**3、操作Zookeeper**

- 启动Zookeeper

```bash
$ bin/zkServer.sh start
```

- 查看进程是否启动

```bash
$ jps
4020 Jps
4001 QuorumPeerMain
```

- 查看状态：

```bash
$ bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: standalone
```

- 停止Zookeeper

```bash
$ bin/zkServer.sh stop
```



### 4.2 集群模式

**1、配置**

- 修改 zoo.cfg 文件

```
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
server.4=hadoop104:2888:3888
# 2888 为集群间通信端口号
# 3888 为选举端口号
# server.2/3/4 为id，不相同即可
```

- 创建 `zkData/myid`，每个机器写不同的，要和前面的对应上

```bash
$ vim zkData/myid
# hadoop102
2
# hadoop103
3
# hadoop104
4
```

- 配置 `bin/zkEnv.sh`

```sh
# 替换前
ZOO_LOG_DIR="."
# 替换后
ZOO_LOG_DIR="/opt/module/zookeeper-3.4.10/logs"

# 添加 JAVA_HOME（远程启动的时候才是必要的，因为会丢失环境变量）
export JAVA_HOME=/opt/module/jdk1.8.0_144
```

- 同步 xsync

**2、启动**

```bash
# 每个机器都要单独启动
./bin/zkServer.sh start
```

仅启动一台机器时：

```
$ ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.
```

启动两台机器（超过半数）：

```
$ ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: leader / follower
```

**3、集群脚本**

```bash
#!/bin/bash

case $1 in
"start"){
        for i in hadoop102 hadoop103 hadoop104
        do
                echo "==== start $i zookeeper ===="
                ssh $i "/opt/module/zookeeper-3.4.10/bin/zkServer.sh start"
        done
};;

"stop"){
        for i in hadoop102 hadoop103 hadoop104
        do
                echo "==== stop $i zookeeper ===="
                ssh $i "/opt/module/zookeeper-3.4.10/bin/zkServer.sh stop"
        done
};;

"status"){
        for i in hadoop102 hadoop103 hadoop104
        do
                echo "==== status $i zookeeper ===="
                ssh $i "/opt/module/zookeeper-3.4.10/bin/zkServer.sh status"
        done
};;

esac
```

### 4.3 客户端操作

- 启动

```bash
$ bin/zkCli.sh
```

- 执行

| 命令基本语法     | 功能描述                                                     |
| ---------------- | ------------------------------------------------------------ |
| help             | 显示所有操作命令                                             |
| ls path [watch]  | 使用 ls 命令来查看当前znode中所包含的内容                    |
| ls2 path [watch] | 查看当前节点数据并能看到更新次数等数据                       |
| create           | 普通创建<br/>-s  含有序列<br/>-e  临时（重启或者超时消失） |
| get path [watch] | 获得节点的值                                                 |
| set              | 设置节点的具体值                                             |
| stat             | 查看节点状态                                                 |
| delete           | 删除节点                                                     |
| rmr              | 递归删除节点                                                 |



## 五、Kakfa 配置

### 5.1 环境准备

- 在 hadoop102 103 104 上均安装 Kafka
- jar 包下载 https://kafka.apache.org/downloads
  - 命名中有两个版本号，第一个为 scala 版本，第二个是 kafka 版本

### 5.2 集群配置

- 修改 `config/server.properties`

```
# broker的全局唯一编号，不能重复
broker.id=0

# 删除topic功能使能（kafka 2.x 版本 不需要配置）
delete.topic.enable=true

# kafka运行时数据存放的路径
log.dirs=/opt/module/kafka_2.12-2.3.1/data

# 配置连接Zookeeper集群地址
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181
```

- 分发安装包到所有物理机上 xsync

- 将 hadoop103 104 上`config/server.properties` 中的 `broker.id=x`进行修改



**单点启动 / 停止** 

依次在 hadoop102、hadoop103、hadoop104 节点上启动/停止 kafka，执行下面的命令

```bash
# 启动
# 前台运行
$ bin/kafka-server-start.sh config/server.properties
# 后台运行
$ bin/kafka-server-start.sh -daemon config/server.properties

# 停止
$ bin/kafka-server-stop.sh
```



**群起 / 群停**

由于 Kafka 中没有给集群启动停止的脚本，需要自己写`kk-all.sh`

需要注意：要先在 `~/.bashrc` 中配置 java 环境变量（xsync）

```bash
# JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin
```

```bash
#!/bin/bash

case $1 in
"start"){
        for i in hadoop102 hadoop103 hadoop104
        do
                echo "==== start $i kafka ===="
                # ssh $i "source /etc/profile"
                ssh $i "/opt/module/kafka_2.12-2.3.1/bin/kafka-server-start.sh -daemon /opt/module/kafka_2.12-2.3.1/config/server.properties"
        done
};;

"stop"){
        for i in hadoop102 hadoop103 hadoop104
        do
                echo "==== stop $i kafka ===="
                # ssh $i "source /etc/profile"
                ssh $i "/opt/module/kafka_2.12-2.3.1/bin/kafka-server-stop.sh"
        done
};;

esac
```

启动/停止 Kafka：（记得先启动 Zookeeper）

```bash
$ ./kk-all.sh start
$ jps
1637 QuorumPeerMain
6071 Kafka
6538 Jps

$ ./kk-all.sh stop
```

### 5.3 命令行操作

- 查看当前服务器中的所有 topic

```bash
$ bin/kafka-topics.sh --zookeeper hadoop102:2181 --list
```

- 创建 topic

```bash
$ bin/kafka-topics.sh --zookeeper hadoop102:2181 --create --replication-factor 3 --partitions 1 --topic first
```

`--replication-factor` 定义副本数；` --partitions` 定义分区数

- 删除 topic

```bash
$ bin/kafka-topics.sh --zookeeper hadoop102:2181 --delete --topic first
```

- 查看某个 Topic 的详情

```bash
$ bin/kafka-topics.sh --zookeeper hadoop102:2181 --describe --topic first
```

- 发送消息

```bash
$ bin/kafka-console-producer.sh --broker-list hadoop102:9092 --topic first
>hello world
>atguigu atguigu
```

- 消费消息

```bash
$ bin/kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --from-beginning --topic first
```

`--from-beginning` 会把 first 主题中以往所有的数据都读取出来



## 六、Spark 配置

### 6.1 环境准备

下载地址：https://archive.apache.org/dist/spark/

有两种版本，hadoop 版下载就能用，不依赖其他组价；without-hadoop 需要依赖已有的 hadoop 组件

> spark-2.3.2-bin-hadoop2.7.tgz
> spark-2.3.2-bin-without-hadoop.tgz 

### 6.2 本地模式 Local

- Jar

解压后进入 Spark 根目录执行（一个算 PI 的程序）：

```bash
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master local[2] \
./examples/jars/spark-examples_2.11-2.3.2.jar 100

bin/spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode client ./examples/jars/spark-examples_2.11-2.3.2.jar 100
```

输出结果：

```bash
# 一大堆迭代过程 
Pi is roughly 3.1424043142404314
```

可以通过访问 http://hadoop102:4040 查看任务运行情况

**「问题」**：运行结束这个页面就关闭了，不能查历史任务执行情况

**「解决」**：添加 Spark History Server

- Spark-shell

进入 Spark-shell，`bin/spark-shell`

```bash
# 创建两个文件，里面输入几行单词
$ vim 1.txt  # xxxxx
$ vim 2.txt  # xxxxx

# 进入 spark-shell 执行
$ bin/spark-shell
> sc.textFile("input").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).collect
```



### 6.3 Standalone 模式

构建一个由 Master + Slave 构成的 Spark 集群，Spark 运行在集群中。

这个要和 Hadoop 中的 Standalone 区别开来.这里的 Standalone 是指只用 Spark 来搭建一个集群, 不需要借助其他的框架.是相对于 Yarn 和 Mesos 来说的.

#### 6.3.1 Spark server配置

1、进入配置文件目录conf，配置spark-evn.sh

```bash
cd conf/
cp spark-env.sh.template spark-env.sh
```

在 `spark-env.sh` 文件中配置如下内容:

```bash
SPARK_MASTER_HOST=hadoop102
SPARK_MASTER_PORT=7077 # 默认端口就是7077, 可以省略不配
```

2、修改 slaves 文件，添加 worker 节点

```bash
cp slaves.template slaves
# 在slaves文件中配置如下内容:
hadoop201
hadoop202
hadoop203
```

3、修改 sbin/spark-config.sh，添加 JAVA_HOME （防止 JAVA_HOME is not set 报错）

```bash
export JAVA_HOME=/opt/module/jdk1.8.0_144
```

4、分发spark-standalone

5、启动 Spark 集群

```bash
sbin/start-all.sh
```

6、网页查看信息：http://hadoop102:8080/

7、测试

```bash
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop102:7077 \
--executor-memory 1G \
--total-executor-cores 6 \
--executor-cores 2 \
./examples/jars/spark-examples_2.11-2.3.2.jar 100
```

#### 6.3.2 spark-history-server

在 Spark-shell 没有退出之前，看到正在执行的任务的日志情况:http://hadoop102:4040. 但是退出之后，执行的所有任务记录全部丢失

所以需要配置任务的历史服务器, 方便在任何需要的时候去查看日志。

- 配置spark-default.conf文件，开启 Log

```bash
cp spark-defaults.conf.template spark-defaults.conf
```

在 `spark-defaults.conf` 文件中, 添加如下内容:

```bash
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://hadoop102:9000/spark-job-log
```

注意:

`hdfs://hadoop201:9000/spark-job-log` 目录必须提前存在, 名字随意

- 修改spark-env.sh文件，添加如下配置

```bash
export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080 -Dspark.history.retainedApplications=30 -Dspark.history.fs.logDirectory=hdfs://hadoop102:9000/spark-job-log"
```

- 分发配置文件
- 启动历史服务
  - 需要先启动 HDFS `$HADOOP_HOME/sbin/start-dfs.sh`
  - 然后再启动: `sbin/start-history-server.sh`

ui 地址: http://hadoop102:18080

### 6.4 Yarn 模式

#### 6.4.1 spark server 配置

- 修改 `${HADOOP_HOME}/etc/hadoop/yarn-site.xml`（仅虚拟机中配置，防止内存不够）

```xml
    <!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默>认是true -->
    <property>
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
    </property>
    <!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默>认是true -->
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>
```

- 修改 `conf/spark-env.sh`，分发

```
YARN_CONF_DIR=/opt/module/hadoop-2.7.2/etc/hadoop
```

- 测试（注意 master、deploy-mode 参数的变化）

```bash
$ start-dfs.sh
$ start-yarn.sh

bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode client \
./examples/jars/spark-examples_2.11-2.3.2.jar 100
```

- spark-shell

```bash
$ bin/spark-shell --master yarn
```

Yarn：http://hadoop103:8088

#### 6.4.2 spark-history-server

`$HADOOP_HOME/etc/hadoop/yarn-site.xml` 中添加

```xml
    <property>
        <name>yarn.log.server.url</name>
        <value>http://hadoop104:19888/jobhistory/logs</value>
        <!-- 填 yarn historyserver 的物理机  -->
    </property>
```

`$SPARK_HOME/conf/spark-defaults.conf`

```xml
spark.yarn.historyServer.address	hadoop102:18080    # spark history Server 物理机
spark.history.ui.port 						18080
spark.eventLog.enabled 						true
spark.eventLog.dir               	hdfs://hadoop102:9000/spark-job-log
spark.history.fs.logDirectory 		hdfs://hadoop102:9000/spark-job-log
```

- 相关服务
  - spark: `sbin/start-all.sh`
  - HDFS: `[hadoop102]$ start-dfs.sh`
  - Yarn: `[hadoop103]$ start-yarn.sh`
  - Yarn-history: `[hadoop104]$ mr-jobhistory-daemon.sh start historyserver`
  - Spark-history: `[hadoop102] $ sbin/start-history-server.sh`

【可以正常展示了】

相关文档解释：[spark深入：配置文件与日志 - Super_Orco - 博客园](https://www.cnblogs.com/sorco/p/7070922.html)

- 查看方式
  - 通过 YARN 查询
    - http://hadoop103:8088/
  - 直接在 spark history server 中查询
    - http://hadoop102:18080/

### 6.5 WordCount 程序

略

## 七、Hive 配置

### 7.1 单机默认配置

下载地址：http://archive.apache.org/dist/hive/

**安装部署：**

- 修改 `conf/hive-env.sh.template` 

```bash
$ mv hive-env.sh.template hive-env.sh
$ vim hive-env.sh
~
~
export HADOOP_HOME=/opt/module/hadoop-2.7.2
export HIVE_CONF_DIR=/opt/module/hive-2.3.0-bin/conf
~
~
```

- hadoop 相关配置

```bash
# 启动 hdfs yarn
$ start-dfs.sh
$ start-yarn.sh

# 创建 hive warehouse（存数据的地方）
$ hadoop fs -mkdir /tmp
$ hadoop fs -mkdir -p /user/hive/warehouse
$ hadoop fs -chmod g+w /tmp
$ hadoop fs -chmod g+w /user/hive/warehouse
```

- Hive 基本操作 

```bash
$ bin/hive
# 查看数据库
hive> show databases;
# 打开默认数据库 
hive> use default;
# 显示 default 数据库中的表 
hive> show tables;
# 创建一张表
hive> create table student(id int, name string);
# 查看表的结构 
hive> desc student;
# 向表中插入数据
hive> insert into student values(1000,"ss");
# 查询表中数据
hive> select * from student;
# 退出 
hive hive> quit;
```

### 7.2 修改默认数据库（derby -> MySQL）

derby 只支持单个客户端连接，仅适用于简单测试。更换成关系型数据库（如MySQL），可支持多客户端连接。

#### 7.2.1 安装 MySQL

- 卸载原有的，安装新的 MySQL 

```bash
# 卸载原有的
$ su -  # 切换到 root 用户
$ rpm -qa | grep mysql 
mysql-libs-5.1.73-7.el6.x86_64
$ rpm -e --nodeps mysql-libs-5.1.73-7.el6.x86_64

# 安装新的 MySQL-server
$ rpm -ivh MySQL-server-5.6.24-1.el6.x86_64.rpm
cat /root/.mysql_secret # 记住默认的root登录密码
OEXaQuS8IWkG19Xs
$ service mysql status
$ service mysql start

# 安装 MySQL-client
$ rpm -ivh MySQL-client-5.6.24-1.el6.x86_64.rpm
# 修改 root 密码
$ mysql -uroot -pOEXaQuS8IWkG19Xs
mysql>SET PASSWORD=PASSWORD('123456');
mysql>exit
```

- 配置 MySQL 远程登录

```bash
$ mysql -uroot -p123456
mysql> use mysql;
mysql> show tables;
mysql> select User, Host, Password from user;

# 修改 user 表，把 Host 表内容修改为%
mysql> update user set host='%' where host='localhost';
# 其他的都删掉
mysql> delete from user where host='hadoop102';
mysql> delete from user where host='127.0.0.1';
mysql> delete from user where host='::1';

mysql> flush privileges;
mysql> quit;

# 都做完切换回原来的用户
```

#### 7.2.2 修改 hive 元数据库

- 拷贝驱动

```bash
$ cp mysql-connector-java-5.1.27-bin.jar /opt/module/hive-2.3.0-bin/lib/
```

- 配置 Metastore 到 MySQL

创建 `conf/hive-site.xml` 

> 官方配置文档
> [AdminManual Metastore Administration - Apache Hive - Apache Software Foundation](https://cwiki.apache.org/confluence/display/Hive/AdminManual+Metastore+Administration)

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?> 
<configuration>
<property> 
	<name>javax.jdo.option.ConnectionURL</name>
	<value>jdbc:mysql://hadoop102:3306/metastore?createDatabaseIfNotExist=true</value>
	<description>JDBC connect string for a JDBC metastore</description>
</property>
<property>
	<name>javax.jdo.option.ConnectionDriverName</name> 	
  <value>com.mysql.jdbc.Driver</value>
	<description>Driver class name for a JDBC metastore</description>
</property>
<property>
	<name>javax.jdo.option.ConnectionUserName</name>
	<value>root</value>
	<description>username to use against metastore database</description>
</property>
<property>
	<name>javax.jdo.option.ConnectionPassword</name> 
  <value>123456</value>
	<description>password to use against metastore database</description>
</property>
</configuration>
```

用于启动 hive-metastore 的配置

```xml
    <property>
         <name>hive.metastore.warehouse.dir</name>
         <value>/user/hive/warehouse</value>
         <description>location of default database for the warehouse</description>
    </property>
		<!-- 可以显示表头列名 -->
    <property>
        <name>hive.cli.print.header</name>
        <value>true</value>
    </property>
		<!-- 显示当前数据库名 -->
    <property>
        <name>hive.cli.print.current.db</name>
        <value>true</value>
    </property>
    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>
    <property>
        <name>datanucleus.schema.autoCreateAll</name>
        <value>true</value>
    </property>
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://hadoop102:9083</value>
    </property>
```

- 启动

```bash
# 初试化hive库
$ bin/schematool -initSchema -dbType mysql
# 启动metastore节点
$ nohup bin/hive --service metastore &
# 启动hiveserver2（可选？） 
$ nohup bin/hive --service hiveserver2 &
# 命令行启动
$ bin/hive
```

### 7.3 Beeline 连接

> Hive学习之路 （四）Hive的连接3种连接方式 - 扎心了，老铁 - 博客园
> https://www.cnblogs.com/qingyunzong/p/8715925.html

### 7.4 常用交互命令

1、`-e`不进入 hive 的交互窗口执行 sql 语句 

```bash
$ bin/hive -e "select id from student;"
```

2、`-f`执行脚本中 sql 语句 

- 创建 hivef.sql 文件

```bash
$ touch hivef.sql
select *from student;
$ bin/hive -f xxx/hivef.sql > xxx/result.txt
```

### 7.5 集成 Tez 引擎

- 解压 tez：`/opt/module/tez-0.9.1-bin`
- 同时上传一个 tez 包到 hdfs，用于给集群中其他节点用

```bash
hadoop fs -put /opt/software/apache-tez-0.9.1-bin.tar.gz/ /tez
```

- 在 HIVE_HOME/conf 下创建 `tez-site.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>tez.lib.uris</name>
        <value>${fs.defaultFS}/tez/apache-tez-0.9.1-bin.tar.gz</value>
    </property>
    <property>
         <name>tez.use.cluster.hadoop-libs</name>
         <value>true</value>
    </property>
    <property>
         <name>tez.history.logging.service.class</name>        
         <value>org.apache.tez.dag.history.logging.ats.ATSHistoryLoggingService</value>
    </property>
</configuration>
```

- 修改 `hive-env.sh`

```bash
# Folder containing extra libraries required for hive compilation/execution can be controlled by:
export TEZ_HOME=/opt/module/tez-0.9.1-bin    #是你的tez的解压目录
export TEZ_JARS=""
for jar in `ls $TEZ_HOME |grep jar`; do
    export TEZ_JARS=$TEZ_JARS:$TEZ_HOME/$jar
done
for jar in `ls $TEZ_HOME/lib`; do
    export TEZ_JARS=$TEZ_JARS:$TEZ_HOME/lib/$jar
done

export HIVE_AUX_JARS_PATH=/opt/module/hadoop-2.7.2/share/hadoop/common/hadoop-lzo-0.4.20.jar$TEZ_JARS
```

- 修改 `hive-site.xml`

```xml
<property>
    <name>hive.execution.engine</name>
    <value>tez</value>
</property>
```

- 测试

```bash
# 启动Hive
$ bin/hive

# 创建表
hive (default)> create table student(
id int,
name string);

# 向表中插入数据
hive (default)> insert into student values(1,"zhangsan");

# 如果没有报错就表示成功了
hive (default)> select * from student;
1       zhangsan
```



## 八、HBase 配置

### 8.1 集群配置

- `conf/hbase-env.sh`

```bash
# 1、注释掉下面两行
# Configure PermSize. Only needed in JDK7. You can safely remove it for JDK8+
export HBASE_MASTER_OPTS="$HBASE_MASTER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m"
export HBASE_REGIONSERVER_OPTS="$HBASE_REGIONSERVER_OPTS -XX:PermSize=128m -XX:MaxPermSiz    e=128m"

# 2、使用单独的 Zookeeper
export HBASE_MANAGES_ZK=false
```

- `conf/hbase-site.sh`

```xml
<!-- 都要根据自己的机器进行配置 -->
		<property>
            <name>hbase.rootdir</name>
            <value>hdfs://hadoop102:9000/HBase</value>
    </property>

    <property>
            <name>hbase.cluster.distributed</name>
            <value>true</value>
    </property>

    <property>
            <name>hbase.zookeeper.quorum</name>
         <value>hadoop102:2181,hadoop103:2181,hadoop104:2181</value>
    </property>

    <property>
            <name>hbase.zookeeper.property.dataDir</name>
         <value>/opt/module/zookeeper-3.4.10/zkData</value>
    </property>
```

- 单独启动

```bash
# 启动 HDFS、Zookeeper
$ start-dfs.sh
$ ${Zookeeper_BASE}/bin/zkServer.sh start  # 每个机器都要单独启动

# HBase
$ bin/hbase-daemon.sh start master   # 在其中一台启动
$ bin/hbase-daemon.sh start regionserver # 都要启动

$ bin/hbase-daemon.sh stop master   # 在其中一台启动
$ bin/hbase-daemon.sh stop regionserver # 都要启动

# 可以在 hadoop102:16010 查看ui界面
```

- 群起 / 群停

```bash
# 启动 HDFS、Zookeeper
$ start-dfs.sh
$ ${Zookeeper_BASE}/bin/zkServer.sh start  # 每个机器都要单独启动

# 配置 conf/regionservers，添加所有 regionserver 的 host
hadoop102
hadoop103
hadoop104

# 群起 / 群停
$ bin/start-hbase.sh
$ bin/stop-hbase.sh

# regionservers
$ bin/hbase-daemons.sh start regionserver
$ bin/hbase-daemons.sh stop regionserver
```

- 进入 shell

```bash
# 如果有 kerberos 认证，需要先 kinit 一下
$ bin/hbase shell
```

### 8.2 常用操作

使用 `help` 查看各种命令使用方式

```bash
# 列出所有指令
> help

# 查看某一组命令的帮助
> help 'COMMAND_GROUP'

# 查看单个命令帮助
> help 'COMMAND'
```

#### 8.2.1 namespace

`Commands: alter_namespace, create_namespace, describe_namespace, drop_namespace, list_namespace, list_namespace_tables`

- list_namespace

```bash
# 查看所有库名
> list_namespace
```

- create_namespace

```bash
> create_namespace 'school'
```

- delete_namespace

```bash
# 注意：只能删除空库
> delete_namespace 'school'
```

- list_namespace_tables

```bash
# 列出库中所有表
> list_namespace_tables 'school'
```

#### 8.2.2 DDL

`Commands: alter, alter_async, alter_status, create, describe, disable, disable_all, drop, drop_all, enable, enable_all, exists, get_table, is_disabled, is_enabled, list, locate_region, show_filters`

- list

```bash
# list 列出所有表
> list
```

- create

```bash
# create '库名:表名', { NAME => '列族名1', 属性名 => 属性值}, {NAME => '列族名2', 属性名 => 属性值}, …
> create 'school:student', {NAME=>'info'}
> create 'school:student', {NAME=>'info', VERSIONS=>5}

# 如果你只需要创建列族，而不需要定义列族属性，那么可以采用以下快捷写法：
# create'表名','列族名1' ,'列族名2', …
# 不写库名，默认 namespace 为 default
> create 'student','info'
```

- desc

```bash
> desc 'student'
```

- disable

```bash
# 停用表，防止对表进行写数据；在修改或删除表之前要 disable
> disable 'student'
> is_disable 'student'
```

- enable

```bash
# 启用表
> enable 'student'
> is_enable 'student'
```

- alter

```bash
# 需要先 disable
> alter 'student', {NAME => 'info', VERSIONS => '5'}
```

- drop

```bash
# 需要先 disable
> drop 'student'
```

- count

```bash
# 查看行数
> count 'student'
```

- truncate

```bash
# 删除表数据
> truncate 'student'
```

#### 8.2.3 DML

` Commands: append, count, delete, deleteall, get, get_counter, get_splits, incr, put, scan, truncate, truncate_preserve`

- scan

```bash
# 查看数据
> scan 'student', {limit => 5}
# 查看每行最近十次修改的数据
> scan 'student', {RAW => true, VERSIONS => 10}
```

- put

```bash
# put '表名', '行键', '列族:列名', '值'
> put 'student', '1001', 'info:name', 'Nick'
```

- get

```bash
> get 'student','1001'
```

- delete

```bash
# 删除某rowkey的全部数据：
> deleteall 'student', '1001'
# 删除某rowkey的某一列数据：
> delete 'student', '1002', 'info:sex'
```

#### 8.2.4 其他操作

- flush

```bash
# 将内存数据落盘
> flush 'student'
```

