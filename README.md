# Hadoop
Hadoop集群搭建

环境：
centos6.6
jdk1.7.0_67
hadoop-2.5.0-cdh5.3.3

一、环境准备
1、搭建3台虚机
2、安装JDK
3、修改主机名
路径：/etc/sysconfig/network
内容：
NETWORKING=yes
HOSTNAME=hadoop0

查看:hostname
如果hostname未修改成指定的，执行：hostname hostname0（根据实际修改）
4、给三台机器分别做好 IP 映射
路径：/etc/hosts
文件末尾追加：
192.168.138.129 hadoop0
192.168.138.130 hadoop1
192.168.138.131 hadoop2
以上2-4三台机器都需要执行

5、SSH免密登录
ssh-keygen -t rsa -P ""
cd ~/.ssh
cat id_rsa.pub >> authorized_keys
测试登录
ssh localhost
exit
以上步骤所以机器都需要执行

复制主机公钥
cp .ssh/id_rsa.pub ~/id_rsa_master.pub

将主机master的公钥复制到slave中
scp -r ~/id_rsa_master.pub hadoop1:~/
scp -r ~/id_rsa_master.pub hadoop2:~/

在hadoop1跟hadoop2中分别执行以下操作：
cat id_rsa_master.pub >> .ssh/authorized_keys

测试：
master机器：
ssh hadoop1
ssh hadoop2
如果不需要密码就可以登录则成功

二、hadoop搭建
1、每台机器创建用户hadoop，密码为hadoop
useradd hadoop
passwd hadoop

2、新建安装目录
mkdir /usr/local/hadoop
并将hadoop的安装包放到文件夹下
解压：tar -xzvf hadoop-2.5.0-cdh5.3.3.tar.gz

3、将安装目录权限赋给hadoop
chown -R hadoop.hadoop /usr/local/hadoop/

4、修改配置文件
路径：/usr/local/hadoop/hadoop-2.5.0-cdh5.3.3/etc/hadoop
a.hadoop-env.sh
export JAVA_HOME=/usr/java/jdk1.7.0_04/
（红色部分为虚拟机本地jdk路径）

b.slaves
文件后追加
hadoop1
hadoop2

c.core-site.xml
添加：
<configuration>
<property>
<name>fs.defaultFS</name>
<value>hdfs://hadoop0:9000</value>
</property>
<property>
<name>hadoop.tmp.dir</name>
<value>file:/usr/local/hadoop/hadoop-tmp</value>
</property>
</configuration>

d.hdfs-site.xml
添加：
<configuration>
<property>
<name>dfs.namenode.name.dir</name>
<value>file:/usr/local/hadoop/hadoop-dfs/name</value>
</property>
<property>
<name>dfs.namenode.data.dir</name>
<value>file:/usr/local/hadoop/hadoop-dfs/data</value>
</property>
<property>
<name>dfs.replication</name>
<value>2</value>
</property>
<property>
<name>dfs.namenode.secondary.http-address</name>
<value>hadoop0:9001</value>
</property>
</configuration>

e.yarn-site.xml
添加：
<configuration>
<property>
<name>yarn.resourcemanager.address</name>
<value>hadoop0:8032</value>
</property>
<property>
<name>yarn.resourcemanager.scheduler.address</name>
<value>hadoop0:8030</value>
</property>
<property>
<name>yarn.resourcemanager.resource-tracker.address</name>
<value>hadoop0:8031</value>
</property>
<property>
<name>yarn.resourcemanager.admin.address</name>
<value>hadoop0:8033</value>
</property>
<property>
<name>yarn.resourcemanager.webapp.address</name>
<value>hadoop0:8088</value>
</property>
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
<property>
<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
<value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
</configuration>

f.mapred-site.xml
<configuration>
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>
<property>
<name>mapreduce.jobhistory.address</name>
<value>hadoop0:10020</value>
</property>
<property>
<name>mapreduce.jobhistory.webapp.address</name>
<value>hadoop0:19888</value>
</property>
</configuration>

5、将 Master 配置好的 Hadoop 系统拷贝到所有 Slave 上
分别在 hadoop1 和 hadoop2 上执行如下命令创建目录：

mkdir -p /usr/local/hadoop
chmod 777 /usr/local/hadoop

复制：
scp -r /usr/local/hadoophadoop-2.5.0-cdh5.3.3/ hadoop@hadoop1:/usr/local/hadoop/
scp -r /usr/local/hadoop/hadoop-2.5.0-cdh5.3.3/ hadoop@hadoop2:/usr/local/hadoop/

6、在 Master 上设置 Hadoop 的环境变量
路径：/etc/profile

export HADOOP_HOME=/usr/local/hadoop/hadoop-2.5.0-cdh5.3.3（红色为你的实际路径）
export PATH=$HADOOP_HOME/bin:$PATH

生效环境变量：source /etc/profile

7、Hadoop 格式化
cd /usr/local/hadoop/hadoop-2.5.0-cdh5.3.3/bin
./hdfs namenode -format

8、Hadoop 启动
cd /usr/local/hadoop/hadoop-2.5.0-cdh5.3.3/sbin
./start-all.sh

启动后查看进程：jps
Master：
50465 ResourceManager
50330 SecondaryNameNode
50933 Jps
50153 DataNode
50062 NameNode
50558 NodeManager

slave：
39637 DataNode
39881 Jps
39729 NodeManager


