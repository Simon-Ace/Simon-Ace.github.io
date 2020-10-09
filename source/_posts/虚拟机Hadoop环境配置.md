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



## 三、运行配置

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
  - 打开 history，在点 log 就能看到任务具体的日志信息了
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
```

