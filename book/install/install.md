####一、flink软件包的下载与解压   
1.从flink官网获得下载链接
1.1进入官网链接：  
http://flink.apache.org
![](images/Snip20161113_50.png) 
1.2进入下载页面：   
http://flink.apache.org/downloads.html
![](images/Snip20161113_50.png) 
1.3获取下载地址：
![](images/Snip20161113_54.png) 
选择相应的版本，下载地址为http://mirrors.tuna.tsinghua.edu.cn/apache/flink/flink-1.1.3/flink-1.1.3-bin-hadoop27-scala_2.10.tgz
2.下载并解压flink
2.1下载命令：
```
wget http://mirrors.tuna.tsinghua.edu.cn/apache/flink/flink-1.1.3/flink-1.1.3-bin-hadoop27-scala_2.10.tgz
```
得到flink-1.1.3-bin-hadoop27-scala_2.10.tgz软件压缩包

2.2解压命令：
```
tar -zxvf flink-1.1.3-bin-hadoop27-scala_2.10.tgz
```
解压得到flink-1.1.3文件夹，文件夹中的内容如下：
![](images/Snip20161113_55.png) 
3.分发flink及其环境变量  
3.1分发flink  
分发命令：  
```
scp -r /bigdata/software/flink-1.1.3  qingcheng12:/bigdata/software/
scp -r /bigdata/software/flink-1.1.3  qingcheng13:/bigdata/software/
```

3.2分发flink的环境变量  
配置FLINK_HOME环境变量  
```
1.编辑环境变量文件
执行命令：
    vim ~/.bashrc
编辑内容：
    在环境办理文件中加入如下内容
    export FLINK_HOME=/bigdata/software/flink-1.1.3
    export PATH=$FLINK_HOME/bin:$PATH
2.分发环境变量文件到其他机器
执行命令：
    scp ~/.bashrc  qingcheng12:~/.bashrc
    scp ~/.bashrc  qingcheng13:~/.bashrc
3.在每个机器上刷新环境变量
执行命令：
    source   ~/.bashrc
4.测试环境环境变量是否配置成功 
执行命令：
    $FLINK_HOME
执行效果：
    出现如下字样说明配置成功
   -bash: /bigdata/software/flink-1.1.3: Is a directory
```

####二、flink在standalone模式主节点下无HA的部署实战
1.部署规划：  
![](images/Snip20161113_56.png) 
2.配置flink-conf.yaml文件
执行命令：
```
vim ${FLINK_HOME}/conf/flink-conf.yaml
```
添加内容：  
    在flink-conf.yaml文件中添加如下内容。
```
# The TaskManagers will try to connect to the JobManager on that host.
jobmanager.rpc.address: qingcheng11

# The heap size for the JobManager JVM
jobmanager.heap.mb: 1024

# The heap size for the TaskManager JVM
taskmanager.heap.mb: 1024

# The number of task slots that each TaskManager offers. Each slot runs one parallel pipeline.
taskmanager.numberOfTaskSlots: 4

# The parallelism used for programs that did not specify and other parallelism.
parallelism.default: 12

# You can also directly specify the paths to hdfs-default.xml and hdfs-site.xml
# via keys 'fs.hdfs.hdfsdefault' and 'fs.hdfs.hdfssite'.
 fs.hdfs.hadoopconf: /bigdata/software/hadoop-2.7.2
```

3.配置slaves文件  
此文件用于指定从节点，一行一个节点.   
执行命令：
```
vim ${FLINK_HOME}/conf/slaves
```
添加内容：  
    在slaves文件中添加如下内容，表示集群的taskManager.
```
qingcheng11
qingcheng12
qingcheng13
```
4.分发配置文件
执行命令：
```
scp -r ${FLINK_HOME}/conf/*  qingcheng12:${FLINK_HOME}/conf/
scp -r ${FLINK_HOME}/conf/*  qingcheng13:${FLINK_HOME}/conf/
```

5.启动flin服务
执行命令：
```
${FLINK_HOME}/bin/start-cluster.sh
```
执行效果：
![](images/Snip20161113_57.png) 

6.验证flink服务   
6.1查看进程验证flink服务，在所有机器上执行，可以看到各自对应的进程名称。   
执行命令：
```
jps
```
执行效果：
![](images/Snip20161113_58.png) 

6.2查看flink的web界面验证服务  
打开网址：  
    在浏览器中打开如下网址
```
http://qingcheng11:8081
```
执行效果：  
集群情况：
![](images/Snip20161113_59.png) 
Job Manager情况：
![](images/Snip20161113_62.png) 
Task Manager情况：
![](images/Snip20161113_61.png) 
可以看出flink集群的整体情况。说明flink在standalone模式下主节点无HA的部署实战是成功的。

7.flink的常用命令
```
1.启动集群
    ${FLINK_HOME}/bin/start-cluster.sh 
2.关闭集群
    ${FLINK_HOME}/bin/stop-cluster.sh
3.启动web客户端
    ${FLINK_HOME}/bin/webclient.sh start
4.关闭web客户端
    ${FLINK_HOME}/bin/webclient.sh stop
5.启动Scala-shell
    ${FLINK_HOME}/bin/start-scala-shell.sh remote qingcheng11 6123
```
       
       
       
       
       
       
       
       
       
1.创建文件夹并上传flink的readme文件
执行命令：
```
hadoop fs -mkdir -p  /input/flink
hadoop fs -put  ${FLINK_HOME}/README.txt  /input/flink/
```
执行效果：
![](images/Snip20161113_63.png)   
   
  
2.打开start-scala-shell.sh
执行命令：
```
${FLINK_HOME}/bin/start-scala-shell.sh remote qingcheng11 6123
```
执行效果：
![](images/Snip20161113_64.png)   
   
3.执行第一个flink程序
执行程序：
```
val file=benv.readTextFile("hdfs://qingcheng11:9000/input/flink/README.txt")
val flinks=file.filter(l =>l.contains("flink"))
flinks.print()
val count=flinks.count
```
shell执行效果：
![](images/Snip20161113_65.png)
web执行效果一：
![](images/Snip20161113_66.png)
web执行效果二：
![](images/Snip20161113_67.png)      