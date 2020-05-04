# SetKylinInHDP

#### 在Hortonworks Sandbox中搭建APACHE KYLIN环境



这两天在Hortonwork Sandbox HDP 3.0.1虚拟机中尝试搭建APACHE KYLIN环境，踩到了很多坑，查了很多中英文的资料，现在记录下来，免得之后遗忘，也供需要的朋友们参考。



kylin版本：apache-kylin-3.0.1-bin-hadoop3

HDP版本：3.0.1



1、Permission denied: user=root, access=WRITE, inode="/kylin":hdfs:hdfs

解决方法：

```bash
$ sudo -u hdfs hdfs dfs -mkdir /kylin
$ sudo -u hdfs hdfs dfs -chmod 777 /kylin
```



2、Something wrong with Hive CLI or Beeline, please execute Hive CLI or Beeline CLI in terminal to find the root cause.

解决方法：

\$ vi bin/find-hive-dependency.sh (第37行)

hive_env=`hive ${hive_conf_properties} -e set 2>&1 | grep 'env:CLASSPATH'` 中的变量 ${hive_conf_properties}去掉（未配置此变量），即修改为

hive_env=`hive -e set 2>&1 | grep 'env:CLASSPATH'` 

![bug2](https://github.com/Jaimejourney/SetKylinInHDP/blob/master/pic/bug2.png)



3、spark not found, set SPARK_HOME, or run bin/download-spark.sh

解决方法：

$ vi /etc/profile

添加如下三行：

export SPARK_HOME=/usr/hdp/current/spark2-client

export HIVE_CONF=/etc/hive/conf

export HCAT_HOME=/usr/hdp/current/hive-webhcat



![bug3](https://github.com/Jaimejourney/SetKylinInHDP/blob/master/pic/bug3.png)

然后再运行$ source /etc/profile

不能直接使用bin/download-spark.sh



4、Caused by: java.lang.NoClassDefFoundError: org/apache/commons/configuration/ConfigurationException

解决方法：将commons-configuration-*.jar 复制到kylin的tomcat/lib 下

$ cp /usr/hdp/share/hst/hst-common/lib/commons-configuration-1.10.jar tomcat/lib/

或者直接用wget下载Maven仓库中的文件也可以



5、kylin启动成功但是打不开web ui界面

![bug5](https://github.com/Jaimejourney/SetKylinInHDP/blob/master/pic/bug5.png)

解决方法：因为Hortonwoks Sandbox的端口是规定好的，默认情况下只有限定的端口可以使用，[具体可以参考官方链接](https://www.cloudera.com/tutorials/hortonworks-sandbox-guide/3.html)，因此我们需要更改kylin的默认端口

1. $ cd /usr/local/apache-kylin-2.6.0-bin/tomcat/conf

2. Open server.xml

![bug51](https://github.com/Jaimejourney/SetKylinInHDP/blob/master/pic/bug51.png)

将默认7070的端口换成任意一个官方推荐的端口（比如4242）就行。