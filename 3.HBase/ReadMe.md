### HBASE安装
- 点击[此链接](https://archive.apache.org/dist/hbase/1.1.2/hbase-1.1.2-bin.tar.gz)下载hbase1.1.2到node-1的BigData中
- 解压
    ```
    tar -xzvf BigData/hbase-1.1.2-bin.tar.gz -C /usr/local/
    cd /usr/local
    mv hbase-1.1.2/ hbase
    ```
- 设置环境变量
    ```
    vim /etc/profile
    # 修改内容
    export HBASE_HOME=/usr/local/hbase
    # 将:$HBASE_HOME/bin加到PATH后面
    export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HBASE_HOME/bin
    
    # 激活PATH
    source /etc/profile
    # 三个服务器都来一下
    ```

- 修改系统变量ulimit
    ```ulimit -n 10240```
- 修改hbase文件
    - hbase-env
        ```
        cd /usr/local/hbase/conf
        vim hbase-env.sh
        #末尾添加以下   
        export JAVA_HOME=/usr/lib/jvm/java/ # java地址
        export HBASE_CLASSPATH=/usr/local/hbase/conf
        # 此配置信息，设置由hbase自己管理zookeeper，不需要单独的zookeeper。
        export HBASE_MANAGES_ZK=true
        export HBASE_HOME=/usr/local/hbase
        export HADOOP_HOME=/usr/local/hadoop
        #Hbase日志目录
        export HBASE_LOG_DIR=/usr/local/hbase/logs
        ```
    - hbase-site.xml
        ```
        vim hbase-site.xml
        # <configuration>替换成
        <configuration>
        	<property>
        		<name>hbase.rootdir</name>
        		<value>hdfs://node-1:9000/hbase</value>
        	</property>
        	<property>
        		<name>hbase.cluster.distributed</name>
        		<value>true</value>
        	</property>
        	<property>
        		<name>hbase.master</name>
        		<value>node-1:60000</value>
        	</property>
        	<property>
        		<name>hbase.zookeeper.quorum</name>
        		<value>node-1,node-2,node-3</value>
        	</property>
        </configuration>
        ```
    - regionservers
        ```
        vim regionservers
        # 内容替换成
        node-1
        node-2
        node-3
        ```
- 复制
    ```
    cd /usr/local/
    scp -r hbase/ node-3:/usr/local
    scp -r hbase/ node-2:/usr/local
    ```
- 运行<br>
    a. 运行hadoop环境<br>
    ```start-all.sh```<br>
    b. 运行hbase环境<br>
    ```
    cd /usr/local/hbase
    bin/start-hbase.sh
    jps
    # 若node-1,node-2,node-3都有HQuorumPeer和HRegionServer说明成功
    ```
#### 简介HBase
HBase是在hdfs基础
  - 特点
    - [ ] 每个值是一个未经解释的字符串，没有数据类型
    - [ ] 用户在表中存储数据，每一行都有一个可排序行键和任意多的列
    - [ ] 表在水平方向由一个或多个列族组成，一个列族中可以包含任意多个列，同一个列族里数据存储在一起
    - [ ] 列族支持动态扩展，可轻松添加一个列族或列，无需预先定义类型数量，所有都以字符串形式存储
    - [ ] HBase更新会生成新版本，保留旧版本
  - 一些定义
    - [ ] 单元格：行，列族，列限定符确定一个单元格，数据存储在单元格里
    - [ ] 列限定符：列族里数据通过列限定符（或列）定位
    - [ ] 时间戳：每个单元格保存同一份数据各个版本，以时间戳索引
  - 定位特点<br>
    四维定位，行键+列族+列限定符+时间戳确定一个单元格位置<br>
        <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/3.HBase/img/1.png?raw=true"/>
#### HBase一些操作
  - 开启环境<br>
    命令行运行
    ```
    # 开启hadoop
    start-all.sh
    # 开启spark
    cd /usr/local/spark
    sbin/start-all.sh
    # 开启hbase
    cd /usr/loacl/hadoop
    ./bin/start-hbase.sh
    ```
  - 建表<br>
    shell里运行./bin/hbase shell（在node-1运行，我的启动速度超慢）<br>
    在shell里运行代码
    ```
    # 创建表格create 表格名，列族名
    create 'student','info'
    
    # 插入数据，按单元格插入
    # put 表名,行键,列族:列,单元格值
    put 'student','1','info:name','Xueqian'
    put 'student','1','info:gender','F'
    put 'student','1','info:age','23'
    
    put 'student','2','info:name','Weiliang'
    put 'student','2','info:gender','M'
    put 'student','2','info:age','24
    
    # 查看表
    scan 'student'
    
    ```
#### 为spark配置hbase
```
cd /usr/local/spark/jars
mkdir hbase
cd hbase
cp /usr/local/hbase/lib/hbase*.jar ./
cp /usr/local/hbase/lib/guava-12.0.1.jar ./
cp /usr/local/hbase/lib/htrace-core-3.1.0-incubating.jar ./
cp /usr/local/hbase/lib/protobuf-java-2.5.0.jar ./

```
- [ ] 点击[此链接](https://akamai.bintray.com/b7/b74385b2cb216d8c2a454b51e93f50175a11cceffb61dc18bcbada434437f935?__gda__=exp=1584943982~hmac=e3a4e804bc653108a7572039f6d8cda54d9d10888191f52dfdd3e817312408d8&response-content-disposition=attachment%3Bfilename%3D%22spark-examples_2.11-1.6.0-typesafe-001.jar%22&response-content-type=application%2Foctet-stream&requestInfo=U2FsdGVkX18DI0wIzXuhA5PunQPuxHcMRehYNtTfDbMFgKgk9ygDiunZBA9Cbu160-5wXER_wF0Qmg2UL8dj2yDG-U1SOsZCJv6UKLhorPQIi_8w_6eiA_9srVu8l_mdhWf61Bnbm5mygE3H72H1XVN6NwvuE-WD7keUUA7woukorgacNzSSN4UaMhfpkSfXCn-zxqKPbl9Vxp1A6LpECrnyNChbXJtVWMMarDGdU_0&response-X-Checksum-Sha1=6beaa6141ad7f622c23f25fb513c243dbac49c7e&response-X-Checksum-Sha2=b74385b2cb216d8c2a454b51e93f50175a11cceffb61dc18bcbada434437f935)下载spark-examples_2.11-1.6.0-typesafe-001.jar到/usr/local/spark/jars/hbase/中
- [ ] 命令行运行
    ```
    cd /usr/local/spark/conf
    vim spark-env.sh
    # 将export SPARK_DIST_CLASSPATH修改成如下
    export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop/bin/hadoop classpath):$(/usr/local/hbase/bin/hbase classpath):/usr/local/spark/jars/hbase/*
    
    # 复制到其他服务器
    scp -r /usr/local/spark/jars/hbase node-2:/usr/local/spark/jars/
    scp -r /usr/local/spark/jars/hbase node-3:/usr/local/spark/jars/
    
    scp /usr/local/spark/conf/spark-env.sh node-2:/usr/local/spark/conf/
    scp /usr/local/spark/conf/spark-env.sh node-3:/usr/local/spark/conf/

    ```

#### 用python读写hbase

  - 读取数据<br>
    在/usr/local/spark/mycode/rdd/新建SparkOperateHbase.py写入如下内容
    ```python
    from pyspark import SparkConf, SparkContext
    conf = SparkConf().setMaster("local").setAppName("ReadHBase")
    sc = SparkContext(conf = conf)
    host = 'localhost'
    table = 'student'
    
    # conf包括服务器地址，表名，
    conf = {"hbase.zookeeper.quorum": host, "hbase.mapreduce.inputtable": table}
    
    # 表类型要转成字符串形式
    keyConv = "org.apache.spark.examples.pythonconverters.ImmutableBytesWritableToStringConverter"
    
    # 值类型要转成字符串形式
    valueConv = "org.apache.spark.examples.pythonconverters.HBaseResultToStringConverter"
    
    # newAPIHadoopRDD-->sparkcontext提供的将表内容以RDD形式加载到spark的api
    # 里面参数（表格格式，指定key类型，定义value格式，key转换类，value转换类，conf=conf）
    hbase_rdd = sc.newAPIHadoopRDD("org.apache.hadoop.hbase.mapreduce.TableInputFormat","org.apache.hadoop.hbase.io.ImmutableBytesWritable","org.apache.hadoop.hbase.client.Result",keyConverter=keyConv,valueConverter=valueConv,conf=conf)
    
    # 数据量
    count = hbase_rdd.count()
    # 持久化
    hbase_rdd.cache()
    output = hbase_rdd.collect()
    # 输出是以(行键,表格内容)的键值对形式
    for (k, v) in output:
            print (k, v)
    
    ```
    运行代码
    ```
    cd /usr/local/spark/mycode/rdd/
    python SparkOperateHbase.py
    ```
  - 写入数据<br>
    在/usr/local/spark/mycode/rdd/新建SparkWriteHBase.py写入如下内容
    ```
    from pyspark import SparkConf, SparkContext
    
    conf = SparkConf().setMaster("local").setAppName("ReadHBase")
    sc = SparkContext(conf = conf)
    host = 'localhost'
    table = 'student'
    
    # key把字符串转成hbase相应类
    keyConv = "org.apache.spark.examples.pythonconverters.StringToImmutableBytesWritableConverter"
    
    # value把字符串转成相应类
    valueConv = "org.apache.spark.examples.pythonconverters.StringListToPutConverter"
    
    # conf包含zookeeper服务器地址，表名，表类型，key类型，值类型
    conf = {"hbase.zookeeper.quorum": host,"hbase.mapred.outputtable": table,"mapreduce.outputformat.class": "org.apache.hadoop.hbase.mapreduce.TableOutputFormat","mapreduce.job.output.key.class": "org.apache.hadoop.hbase.io.ImmutableBytesWritable","mapreduce.job.output.value.class": "org.apache.hadoop.io.Writable"}
    
    # 按单元格插入
    rawData = ['3,info,name,Rongcheng','3,info,gender,M','3,info,age,26','4,info,name,Guanhua','4,info,gender,M','4,info,age,27']
    sc.parallelize(rawData).map(lambda x: (x[0],x.split(','))).saveAsNewAPIHadoopDataset(conf=conf,keyConverter=keyConv,valueConverter=valueConv)
    
    ```
    运行代码
    ```
    cd /usr/local/spark/mycode/rdd/
    python SparkOperateHbase.py
    ```