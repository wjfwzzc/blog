---
layout: post
title: 搭建Hadoop平台的一些笔记
date: 2016-01-04 20:12:43
category: distribute system
tags: hadoop
comments: true
---
现在搭建Hadoop集群已经没有以前那么麻烦了，以下只是一个简单的记录。环境为Ubuntu 14.04 LTS 64位。

##准备工作
首先要安装JDK，Ubuntu党可以从`ppa:webupd8team/java`安装，配置环境变量，老生常谈不再多叙。

其次要配置一下ssh，多节点间各机器使用ssh免登录。单机伪分布式就把自己的设置一下就好了。

hosts要注意，如果主机在内网中，那么hosts里`127.0.1.1`那个映射需要改为你的内网地址。这个地方设错了可能导致后面Yarn运行异常。

##配置Hadoop本地模式
Hadoop可以从[Apache官网](http://hadoop.apache.org/releases.html)下载，我使用了2.6.3版本，直接下载了binary。

下载后解压到用户目录下，将目录更名为`hadoop`。以下操作的工作目录都为此目录。

所有的配置文件都在`etc/hadoop/`下。配置文件都是xml格式的。

有些主机可能需要做的一步是，在`hadoop-env.sh`中修改`export JAVA_HOME=/usr/lib/jvm/java-8-oracle`。也就是把JDK地址显式地写上。

这时应该已经可以在本地模式下运行Hadoop了。我们可以测试一下。
{% highlight bash linenos %}
mkdir input
cp etc/hadoop/*.xml input
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.3.jar wordcount input output
{% endhighlight %}

程序运行结束后我们运行{% highlight bash %} cat output/* {% endhighlight %}查看一下结果就可以了。

##配置Hadoop伪分布模式
运行以下命令建立若干目录备用{% highlight bash %} mkdir tmp hdfs hdfs/name hdfs/data {% endhighlight %}

首先配置HDFS。在`etc/hadoop/core-site.xml`的`configuration`标签中添加如下配置项：
{% highlight xml linenos %}
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/home/wjfwzzc/hadoop/tmp</value>
    </property>
<!--
    <property>
        <name>io.file.buffer.size</name>
        <value>131702</value>
    </property>
-->
</configuration>
{% endhighlight %}

注释掉的配置项的含义大都直接可从名字中看出来，看心情添加，下同。

修改`etc/hadoop/hdfs-site.xml`文件：
{% highlight xml linenos %}
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/home/wjfwzzc/hadoop/hdfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/home/wjfwzzc/hadoop/hdfs/data</value>
    </property>
<!--
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>localhost:9001</value>
    </property>
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
-->
</configuration>
{% endhighlight %}

这里主要定义了本机NameNode和DataNode的状态。

现在可以格式化NameNode了。运行{% highlight bash %} bin/hdfs namenode -format {% endhighlight %}

格式化结束后就可以启动集群了。运行{% highlight bash %} sbin/start-dfs.sh {% endhighlight %}启动成功后打开`http://localhost:50070/`，你应当可以看到Hadoop的管理界面。

这时你也可以运行wordcount测试一下。首先要把文件上传到hdfs上去，后面则是一致的
{% highlight bash linenos %}
bin/hdfs dfs -mkdir /input
bin/hdfs dfs -put etc/hadoop/*.xml /input
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.3.jar wordcount /input /output
{% endhighlight %}

运行结束后运行{% highlight bash %} bin/hdfs dfs -cat /output/* {% endhighlight %}查看结果。注意这里的地址是hdfs上的地址。

##配置Yarn
复制`etc/hadoop/mapred-site.xml.template`到`etc/hadoop/mapred-site.xml`，打开进行编辑：
{% highlight xml linenos %}
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
<!--
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>localhost:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>localhost:19888</value>
    </property>
-->
</configuration>
{% endhighlight %}

然后修改配置文件`etc/hadoop/yarn-site.xml`：
{% highlight xml linenos %}
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
<!--
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>localhost:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>localhost:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>localhost:8031</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>localhost:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>localhost:8088</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>768</value>
    </property>
-->
</configuration>
{% endhighlight %}

接下来就可以启动Yarn了，运行{% highlight bash %} sbin/start-yarn.sh {% endhighlight %}启动成功后打开`http://localhost:8088/`，你应当可以看到Yarn的管理界面。

这时你运行wordcount，它应当是运行在Yarn上的。运行方式与之前完全一致。

到此已经搭建起一套完整的Hadoop+Yarn伪分布式集群，已经可以进行日常的Map-Reduce程序开发测试。增加机器只需要修改`etc/hadoop/slaves`文件`，并配置好各机器间的网络配置即可。

##计划中的下一步
配置Spark，待续。