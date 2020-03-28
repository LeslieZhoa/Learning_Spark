### Spark Streaming
- 数据类型：
    - 静态数据
        - [ ] 特点：批量处理
        - [ ] 传统数据处理流程：
            1. 数据采集存储
            2. 用户主动发出查询获取结果
    - 流数据
        - [ ] 特点：
            1. 数据快速持续到达，潜在大小也许无穷
            2. 数据来源众多，格式复杂
            3. 数据量大，一旦处理完，要么丢弃，要么归档处理
            4. 注重数据整体价值，不过分关注个别数据
            5. 数据顺序颠倒或不完整，系统无法控制将要处理的新到达数据顺序
        - [ ] 实时计算
        - [ ] 数据价值随时间流逝降低
        - [ ] 要求：高性能，实时性，分布性
        - [ ] 过程：
            1. 数据实时采集
            2. 数据实时计算
            3. 主动发起查询
- Strom
    - 特点：
        - [ ] 可毫秒级实时计算
        - [ ] 高度容错
        - [ ] 实时流计算
- Spark Streaming
    - 特点：
        - [ ] 模仿流计算小批量处理
        - [ ] 可实现秒级响应，变相实现高效流计算
        - [ ] 更高效容错处理
        - [ ] 兼容批量实时
    - 数据抽象
        - [ ] Spark Core -->数据抽象：RDD
        - [ ] Spark SQL -->数据抽象：DataFrame
        - [ ] Spark Streaming -->数据抽象：DStream
    - DStream:<br>
        即为一系列RDD集合
        - [ ] DStream操作：<br>
            1. 在Spark Streaming中会有组件Receiver，作为一个长期运行的task跑在Execution上
            2. 每个receiver（包括套接字流，文件流，kafka读取数据流）单独接受一个数据源Input DStream
            3. Spark Streaming通过input DStream与外部数据源进行连接，读取相关数据
        - [ ] 编写Spark Streaming程序
            1. 通过创建DStream来定义输入源
            2. 通过对DStream应用转换操作和输出操作定义流计算
            3. 用streamingContext.start()来开始接受数据和处理流程
            4. 通过streamingContext.awaitTermination()等待处理结束（手动结束或因错误结束）
            5. 通过streamingContext.stop()手动结束流计算
        - [ ] 运行Spark Streaming首先生成StreamingContext对象--> 主入口
            1. pyspark环境下
                ```python
                from pyspark.streaming import StreamingContext
                StreamingContext(sc, 1)
                ```
            2. python下
                ```python
                from pyspark import SparkContext, SparkConf
                from pyspark.streaming import StreamingContext
                conf = SparkConf()
                conf.setAppName('TestDStream')
                conf.setMaster('local[2]')
                sc = SparkContext(conf = conf)
                ssc = StreamingContext(sc, 1)

                ```
- 基本数据源获取SparkStreaming
    - 文件流
        - [ ] 创建文件
            ```
            # shell 下运行
            cd /usr/local/spark/mycode
            mkdir streaming
            cd streaming
            mkdir logfile
            
            # 将该github中5.Spark Streaming下的file.txt放置到node-1的/usr/local/spark/mycode/streaming中
            # file.txt是取自https://github.com/fighting41love/funNLP/blob/master/data/%E6%95%8F%E6%84%9F%E8%AF%8D%E5%BA%93/%E8%89%B2%E6%83%85%E8%AF%8D%E5%BA%93.txt该目录下的色情词库
            ```
            ```python
            vim WriteFile.py
            # 在WriteFile.py写入如下代码,不断生成文本文件看作流源
            import numpy as np
            import time
            import os
            
            with open('/usr/local/spark/mycode/streaming/file.txt','r',encoding='utf-8') as f:
                lines = f.readlines()
            index = np.random.randint(low=0,high=len(lines)-10)
            newlines = lines[index:index+10]
            w_length = np.random.randint(low=5,high=15)
            file_index = 1
            
            write_base = '/usr/local/spark/mycode/streaming/logfile'
            
            if not os.path.exists(write_base):
                os.makedirs(write_base)
            while file_index < 1000:
                
                with open('%s/%d.txt'%(write_base,file_index),'w') as f:
                    for _ in range(w_length):
                        f.write(newlines[np.random.randint(low=0,high=len(newlines))].strip())
                        f.write(' ')
                file_index += 1
                index = np.random.randint(low=0,high=len(lines)-10)
                newlines = lines[index:index+10]
                w_length = np.random.randint(low=5,high=15)
                time.sleep(5)
            
            ```
        - [ ] 写入pyspark
            开启hadoop，spark环境，具体操作可参考前几节
            ```python
            # 运行pyspark，在pyspark写入如下代码
            from pyspark import SparkContext
            from pyspark.streaming import StreamingContext
            ssc = StreamingContext(sc, 10)
            lines = ssc. \
            ... textFileStream('file:///usr/local/spark/mycode/streaming/logfile')
            words = lines.flatMap(lambda line: line.split(' '))
            wordCounts = words.map(lambda x : (x,1)).reduceByKey(lambda a,b:a+b)
            wordCounts.pprint()
            ssc.start()
            ssc.awaitTermination()
            ```
            ```
            # 新打开个终端窗口来生成文件流
            cd /usr/local/spark/mycode/streaming/
            python WriteFile.py
            
            # 在pyspark窗口就可以看到词频统计

            ```
        - [ ] 采用独立应用程序方式
            ```python
            cd /usr/local/spark/mycode/streaming/
            vim FileStreaming.py
            # FileStreaming.py里面写入如下代码
            from pyspark import SparkContext, SparkConf
            from pyspark.streaming import StreamingContext
            
            conf = SparkConf()
            conf.setAppName('TestDStream')
            conf.setMaster('local[2]')
            sc = SparkContext(conf = conf)
            ssc = StreamingContext(sc, 10)
            lines = ssc.textFileStream('file:///usr/local/spark/mycode/streaming/logfile')
            words = lines.flatMap(lambda line: line.split(' '))
            wordCounts = words.map(lambda x : (x,1)).reduceByKey(lambda a,b:a+b)
            wordCounts.pprint()
            ssc.start()
            ssc.awaitTermination()
            ```
            ```
            # WriteFile.py别kill掉提交FileStreaming.py
            cd /usr/local/spark/mycode/streaming/
            /usr/local/spark/bin/spark-submit FileStreaming.py
            
            # 在窗口可以看到词频统计
            ```
    - 套接字流<br>
        socket一直处于阻塞状态，等待响应
        - [ ] 使用nc程序
            ```python
            cd /usr/local/spark/mycode/streaming/
            vim NetworkWordCount.py
            # 在NetworkWordCount.py内写入如下代码
            from __future__ import print_function
            import sys
            from pyspark import SparkContext
            from pyspark.streaming import StreamingContext
            
            if __name__ == "__main__":
                if len(sys.argv) != 3:
                    print("Usage: NetworkWordCount.py <hostname> <port>", file=sys.stderr)
                    exit(-1)
                sc = SparkContext(appName="PythonStreamingNetworkWordCount")
                ssc = StreamingContext(sc, 1)
                
                # socketTextStream-->定义套接字流，接入参数为网址和端口号 
                lines = ssc.socketTextStream(sys.argv[1], int(sys.argv[2]))
                counts = lines.flatMap(lambda line: line.strip().split(" ")) \
                              .map(lambda word: (word, 1)) \
                              .reduceByKey(lambda a, b: a+b)
                # counts.collect()
                
                counts.pprint()
                ssc.start()
                ssc.awaitTermination()
            ```
            打开nc端口
            ```
            nc -lk 9999
            ```
            运行程序
            ```
            # 另起端口
            cd /usr/local/spark/mycode/streaming/
            /usr/local/spark/bin/spark-submit  --master local[2] NetworkWordCount.py localhost 9999
            
            # 在nc那个终端窗口随便写点啥，回车就是一行，在运行程序端口可以看到统计词频
            ```
        - [ ] 使用socket编程自定义数据源
            ```python
            cd /usr/local/spark/mycode/streaming/
            vim DataSourceSocket.py
            # 在DataSourceSocket.py内写入如下代码,这是一个socket程序
            import socket
            # 生成socket对象
            server = socket.socket()
            # 绑定ip和端口
            server.bind(('localhost', 9999))
            # 监听绑定的端口
            server.listen(1)
            while 1:
                # 为了方便识别，打印一个“我在等待”
                print("I'm waiting the connect...")
                # 这里用两个值接受，因为连接上之后使用的是客户端发来请求的这个实例
                # 所以下面的传输要使用conn实例操作
                conn,addr = server.accept()
                # 打印连接成功
                print("Connect success! Connection is from %s " % addr[0])
                # 打印正在发送数据
                print('Sending data...')
                conn.send('I love hadoop I love spark hadoop is good spark is fast'.encode())
                conn.close()
                print('Connection is broken.')

            ```
            运行
            ```
            cd /usr/local/spark/mycode/streaming/
            # 启动socket
            /usr/local/spark/bin/spark-submit DataSourceSocket.py
            # 另起终端启动NetworkWordCount
            cd /usr/local/spark/mycode/streaming/
            /usr/local/spark/bin/spark-submit  --master local[2] NetworkWordCount.py localhost 9999
            # 在该终端可看见词频统计
            ```
    - RDD队流<br>
        功能：每隔1秒创建一个RDD，Streaming每隔2秒就对数据进行处理
        ```python
        cd /usr/local/spark/mycode/streaming/
        vim RDDQueueStream.py
        # RDDQueueStream.py下写以下代码
        import time
        from pyspark import SparkContext
        from pyspark.streaming import StreamingContext
        
        if __name__ == "__main__":
            sc = SparkContext(appName="PythonStreamingQueueStream")
            ssc = StreamingContext(sc, 2)
            #创建一个队列，通过该队列可以把RDD推给一个RDD队列流
            rddQueue = []
            for i in range(5):
                rddQueue += [ssc.sparkContext.parallelize([j for j in range(1, 1001)], 10)]
                time.sleep(1)
            #创建一个RDD队列流
            inputStream = ssc.queueStream(rddQueue)
            mappedStream = inputStream.map(lambda x: (x % 10, 1))
            reducedStream = mappedStream.reduceByKey(lambda a, b: a + b)
            reducedStream.pprint()
            ssc.start()
            # 队列处理完毕结束
            ssc.stop(stopSparkContext=True, stopGraceFully=True)

        ```
        运行
        ```
        cd /usr/local/spark/mycode/streaming/
        /usr/local/spark/bin/spark-submit --master local[2] RDDQueueStream.py
        ```
- Kafka数据源
    - 特点：
        - [ ] 高吞吐量，分布式发布订阅消息系统
        - [ ] 支持在线实时和批量离线处理
        - [ ] 不同类型分布式系统可以统一接入
        - [ ] 实现和hadoop各个组件之间不同类型数据的实时高效交换
    - 组件：
        - [ ] Broker:一个到多个服务器
        - [ ] Topic:每条发布到kafka集群消息都有一个类别，这个类别为Topic
        - [ ] parttition:每个topic包含一个或多个partition
        - [ ] producer:发布消息
        - [ ] consumer:读取消息
        - [ ] consumer Group:每个consumer只属于宇哥consumer group，若不指定group name，则属于默认group
    kafka运行依赖于zookeeper；topic,consumer,partition,broker等注册信息都储存在zookeeper中
    - 安装kafka集群
        - [ ] 点击[此链接](https://kafka.apache.org/downloads)，一直往下翻翻到Scala 2.11  - kafka_2.11-0.10.1.0.tgz (asc, md5)下载放到node-1的~/BigData下
        - [ ] 解压 
            ```
            tar -zxf  ~/BigData/kafka_2.11-0.10.1.0.tgz -C /usr/local
            cd /usr/local
            sudo mv kafka_2.11-0.10.1.0/ ./kafka
            
            ```
        - [ ] 配置文件
            ```
            cd /usr/local/kafka
            mkdir log
            cd log
            mkdir kafka
            mkdir zookeeper
            cd ..
            mkdir zookeeper
            vim zookeeper/myid
            # 里面写入0
            ```
            修改zookeeper.propertics
            ```
            cd /usr/local/kafka/config
            vim zookeeper.propertics
            # 里面添加和修改如下
            dataDir=/usr/local/kafka/zookeeper
            dataLogDir=/usr/local/kafka/log/zookeeper
            # the port at which the clients will connect
            clientPort=2182
            # disable the per-ip limit on the number of connections since this is a non-production config
            maxClientCnxns=100
            maxClientCnxns=100
            
            ticketTime=2000
            
            initLimit=10
            
            syncLimit=5
            
            server.0=192.168.199.128:2888:3888
            
            server.1=192.168.199.129:2888:3888
            
            server.2=192.168.199.130:2888:3888
            
            ```
            修改server.properties
            ```
            cd /usr/local/kafka/config
            vim server.properties
            # 里面增加和修改如下
            # Licensed to the Apache Software Foundation (ASF) under one or more
            # contributor license agreements.  See the NOTICE file distributed with
            # this work for additional information regarding copyright ownership.
            # The ASF licenses this file to You under the Apache License, Version 2.0
            # (the "License"); you may not use this file except in compliance with
            # the License.  You may obtain a copy of the License at
            #
            #    http://www.apache.org/licenses/LICENSE-2.0
            #
            # Unless required by applicable law or agreed to in writing, software
            # distributed under the License is distributed on an "AS IS" BASIS,
            # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
            # See the License for the specific language governing permissions and
            # limitations under the License.
            
            # see kafka.server.KafkaConfig for additional details and defaults
            
            ############################# Server Basics #############################
            
            # The id of the broker. This must be set to a unique integer for each broker.
            broker.id=0
            
            # Switch to enable topic deletion or not, default value is false
            #delete.topic.enable=true
            
            ############################# Socket Server Settings #############################
            
            # The address the socket server listens on. It will get the value returned from 
            # java.net.InetAddress.getCanonicalHostName() if not configured.
            #   FORMAT:
            #     listeners = security_protocol://host_name:port
            #   EXAMPLE:
            #     listeners = PLAINTEXT://your.host.name:9092
            #listeners=PLAINTEXT://:9092
            
            # Hostname and port the broker will advertise to producers and consumers. If not set, 
            # it uses the value for "listeners" if configured.  Otherwise, it will use the value
            # returned from java.net.InetAddress.getCanonicalHostName().
            #advertised.listeners=PLAINTEXT://your.host.name:9092
            
            # The number of threads handling network requests
            port=9092
            hostname=192.168.199.128
            
            log.dirs=/usr/local/kafka/log/kafka
            num.network.threads=3
            
            # The number of threads doing disk I/O
            num.io.threads=8
            
            # The send buffer (SO_SNDBUF) used by the socket server
            socket.send.buffer.bytes=102400
            
            # The receive buffer (SO_RCVBUF) used by the socket server
            socket.receive.buffer.bytes=102400
            
            # The maximum size of a request that the socket server will accept (protection against OOM)
            socket.request.max.bytes=104857600
            
            
            ############################# Log Basics #############################
            
            # A comma seperated list of directories under which to store log files
            log.dirs=/tmp/kafka-logs
            
            # The default number of log partitions per topic. More partitions allow greater
            # parallelism for consumption, but this will also result in more files across
            # the brokers.
            num.partitions=1
            
            # The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
            # This value is recommended to be increased for installations with data dirs located in RAID array.
            num.recovery.threads.per.data.dir=1
            
            ############################# Log Flush Policy #############################
            
            # Messages are immediately written to the filesystem but by default we only fsync() to sync
            # the OS cache lazily. The following configurations control the flush of data to disk.
            # There are a few important trade-offs here:
            #    1. Durability: Unflushed data may be lost if you are not using replication.
            #    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
            #    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to exceessive seeks.
            # The settings below allow one to configure the flush policy to flush data after a period of time or
            # every N messages (or both). This can be done globally and overridden on a per-topic basis.
            
            # The number of messages to accept before forcing a flush of data to disk
            #log.flush.interval.messages=10000
            
            # The maximum amount of time a message can sit in a log before we force a flush
            #log.flush.interval.ms=1000
            
            ############################# Log Retention Policy #############################
            
            # The following configurations control the disposal of log segments. The policy can
            # be set to delete segments after a period of time, or after a given size has accumulated.
            # A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
            # from the end of the log.
            
            # The minimum age of a log file to be eligible for deletion
            log.retention.hours=168
            
            # A size-based retention policy for logs. Segments are pruned from the log as long as the remaining
            # segments don't drop below log.retention.bytes.
            #log.retention.bytes=1073741824
            
            # The maximum size of a log segment file. When this size is reached a new log segment will be created.
            log.segment.bytes=1073741824
            
            # The interval at which log segments are checked to see if they can be deleted according
            # to the retention policies
            log.retention.check.interval.ms=300000
            
            ############################# Zookeeper #############################
            
            # Zookeeper connection string (see zookeeper docs for details).
            # This is a comma separated host:port pairs, each corresponding to a zk
            # server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
            # You can also append an optional chroot string to the urls to specify the
            # root directory for all kafka znodes.
            zookeeper.connect=192.168.199.128:2182,192.168.199.129:2182,192.168.199.130:2182
            
            # Timeout in ms for connecting to zookeeper
            zookeeper.connection.timeout.ms=6000

            ```
        - 克隆
            ```
            cd /usr/local/
            scp -r kafka node-2:/usr/local/
            scp -r kafka node-3:/usr/local/
            ```
            - [ ] 修改参数<br>
                node-2
                ```
                cd /usr/local/kafka/config/
                vim server.properties
                # 修改如下
                broker.id=1
    
                hostname=192.168.199.129
                ```
                ```
                cd /usr/local/kafka/zookeeper/
                vim myid
                # 把0修改成1
                ```
                node-3
                ```
                cd /usr/local/kafka/config/
                vim server.properties
                # 修改如下
                broker.id=2
    
                hostname=192.168.199.130
                ```
                ```
                cd /usr/local/kafka/zookeeper/
                vim myid
                # 把0修改成2
                ```
    - 测试
        - [ ] 启动zookeeper
            ```
            # 在三个服务器每个都运行
            cd /usr/local/kafka
            ./bin/zookeeper-server-start.sh config/zookeeper.properties
            # 窗口不要关，另起其他端口
            ```
        - [ ] 启动kafka
            ```
             # 另起窗口在三个服务器每个都运行
            cd /usr/local/kafka
            bin/kafka-server-start.sh config/server.properties
            # 窗口不要关，另起其他端口
            ```
        - [ ] 创建名为“wordsendertest”的Topic
            ```
            # 另起窗口在node-1运行
            cd /usr/local/kafka
            bin/kafka-topics.sh --create --zookeeper 192.168.199.128:2182,192.168.199.129:2182,192.168.199.130:2182 --replication-factor 1 --partitions 1 --topic wordsendertest
            
            # 可以用list列出所有创建的Topic，验证是否创建成功
            ./bin/kafka-topics.sh  --list  --zookeeper  192.168.199.128:2182,192.168.199.129:2182,192.168.199.130:2182
            ```
        - [ ] 用生产者（Producer）来产生一些数据
            ```
            cd /usr/local/kafka
            bin/kafka-console-producer.sh --broker-list 192.168.1.128:9092,192.168.1.129:9092,192.168.1.130:9092 --topic wordsendertest
            #可在终端写入一些文本如
            
            # hello hadoop
            # hello spark
            ```
        - [ ] 启动一个消费者查看数据
            ```
            # 另起终端
            cd /usr/local/kafka
            bin/kafka-console-consumer.sh --zookeeper 192.168.1.128:2182,192.168.1.129:2182,192.168.1.130:2182 --topic wordsendertest --from-beginning
            # 可以看到刚刚写的数据
            ```
    - 添加jar包<br>
        - [ ] 创建目录
            ```
            cd /usr/local/spark/jars
            mkdir kafka

            ```
        - [ ] 复制jar包
            1. 点击[此链接](https://javalibs.com/artifact/org.apache.spark/spark-streaming-kafka-0-8_2.11)找到spark-streaming-kafka-0-8_2.11-2.4.5.jar下载放置到node-1的/usr/local/spark/jars/kafka下;其中2.11对应scala版本，2.45对应spark版本
            2. 复制kafka的jar包
                ```
                cd /usr/local/kafka/libs
                cp ./* /usr/local/spark/jars/kafka

                ```
        - [ ] 修改Spark配置文件
            ```
            cd  /usr/local/spark/conf
            vim spark-env.sh
            # 修改以下内容
            export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop/bin/hadoop classpath):$(/usr/local/hbase/bin/hbase classpath):/usr/local/spark/jars/hbase/*:/usr/local/spark/examples/jars/*:/usr/local/spark/jars/kafka/*:/usr/local/kafka/libs/*
            
            ```
        - [ ] 复制<br>
            其他服务器也做完全一直操作或直接复制修改文件
    - 编写Spark Streaming程序使用Kafka数据源
        ```python
        cd /usr/local/spark/mycode/streaming/
        vim KafkaWordCount.py
        # KafkaWordCount.py里写入如下内容
        from __future__ import print_function
        import sys
        from pyspark import SparkContext
        from pyspark.streaming import StreamingContext
        from pyspark.streaming.kafka import KafkaUtils 
        
        if __name__ == "__main__":
            if len(sys.argv) != 3:
                print("Usage: KafkaWordCount.py <zk> <topic>", file=sys.stderr)
                exit(-1)
            sc = SparkContext(appName="PythonStreamingKafkaWordCount")
            ssc = StreamingContext(sc, 1)
            
            # zookeeper地址和topic
            zkQuorum, topic = sys.argv[1:]
            
            # 创建kafka流
            kvs = KafkaUtils. \
                createStream(ssc, zkQuorum, "spark-streaming-consumer", {topic: 1})
            lines = kvs.map(lambda x: x[1])
            counts = lines.flatMap(lambda line: line.split(" ")) \
                .map(lambda word: (word, 1)) \
                .reduceByKey(lambda a, b: a+b)
            counts.pprint()
            ssc.start()
            ssc.awaitTermination()
        
        ```
        运行
        ```
        cd /usr/local/spark/mycode/streaming/
        /usr/local/spark/bin/spark-submit --master local[2] ./KafkaWordCount.py 192.168.199.128:2182,192.168.199.129:2182,192.168.199.130:2182 wordsendertest
        # 终端可看词频结果

        ```
    
            
- 转换操作
    - 无状态转换
        直接对流计算不进行累加
        - [ ] map(func)
        - [ ] flatMap(func)
        - [ ] filter(func)
        - [ ] repartition(numPartitions)-->重分区
        - [ ] reduce(func)-->聚合函数，聚合运算
        - [ ] count()
        - [ ] union(otherStream)-->返回合并的新DStream
        - [ ] countByValue()
        - [ ] reduceByKey(func,[numTasks])
        - [ ] .join(otherStream,[numTasks])
        - [ ] cogroup(otherStream,[numTasks])-->将(K,V)键值对和(K,W)键值对转换成(K,seq(V),seq(W))
        - [ ] transform(func)-->对源DStream每个RDD应用函数，创建新的DStream
    - 有状态转换
        会对流进行一些累加操作
        - [ ] window(windowLength, slideInterval)--> 窗口化算得到一个新的Dstream
        - [ ] countByWindow(windowLength, slideInterval) 返回流中元素的一个滑动窗口数
        - [ ] reduceByWindow(func, windowLength, slideInterval)--> 返回一个单元素流。利用函数func聚集滑动时间间隔的流的元素创建这个单元素流。函数func必须满足结合律，从而可以支持并行计算
        - [ ] reduceByKeyAndWindow(func, windowLength, slideInterval, [numTasks])--> 应用到一个(K,V)键值对组成的DStream上时，会返回一个由(K,V)键值对组成的新的DStream。每一个key的值均由给定的reduce函数(func函数)进行聚合计算。注意：在默认情况下，这个算子利用了Spark默认的并发任务数去分组。可以通过numTasks参数的设置来指定不同的任务数
        - [ ] reduceByKeyAndWindow(func, invFunc, windowLength, slideInterval, [numTasks])--> 更加高效的reduceByKeyAndWindow，invFunc为func逆向操作，对离开窗口数据逆向操作，新加数据正向操作，减少重复计算窗口数据计算量
            - 小例子
                ```python
                cd /usr/local/spark/mycode/streaming/
                vim WindowedNetworkWordCount.py
                # WindowedNetworkWordCount.py内写入如下内容
                from __future__ import print_function
                import sys
                from pyspark import SparkContext
                from pyspark.streaming import StreamingContext
                if __name__ == "__main__":
                    if len(sys.argv) != 3:
                        print("Usage: WindowedNetworkWordCount.py <hostname> <port>", file=sys.stderr)
                        exit(-1)
                    sc = SparkContext(appName="PythonStreamingWindowedNetworkWordCount")
                    ssc = StreamingContext(sc, 10)
                    # 加个断点保存 ssc.checkpoint("file:///usr/local/spark/mycode/streaming/socket/checkpoint")
                    lines = ssc.socketTextStream(sys.argv[1], int(sys.argv[2]))
                    counts = lines.flatMap(lambda line: line.split(" "))\
                                  .map(lambda word: (word, 1))\
                                  . reduceByKeyAndWindow(lambda x, y: x + y, lambda x, y: x - y, 30, 10)
                    counts.pprint()
                    ssc.start()
                    ssc.awaitTermination()
    
                ```
                运行nc程序
                ```
                nc -lk 9999
                ```
                运行wordcount
                ```
                # 另起终端
                cd /usr/local/spark/mycode/streaming/
                /usr/local/spark/bin/spark-submit --master local[2] WindowedNetworkWordCount.py localhost 9999
                # 在nc的窗口输入字符，wordcount窗口就可以显示统计词频
                ```
        - [ ] updateStateByKey-->累加之前所有流数据
            - 小例子:
                ```python
                cd /usr/local/spark/mycode/streaming/
                vim NetworkWordCountStateful.py 
                # 里面写入如下代码
                from __future__ import print_function
                import sys
                from pyspark import SparkContext
                from pyspark.streaming import StreamingContext
                if __name__ == "__main__":
                    if len(sys.argv) != 3:
                        print("Usage: NetworkWordCountStateful.py <hostname> <port>", file=sys.stderr)
                        exit(-1)
                    sc = SparkContext(appName="PythonStreamingStatefulNetworkWordCount")
                    ssc = StreamingContext(sc, 1)
                    ssc.checkpoint("file:///usr/local/spark/mycode/streaming/stateful/") 
                    # RDD with initial state (key, value) pairs
                    initialStateRDD = sc.parallelize([(u'hello', 1), (u'world', 1)])
                
                    def updateFunc(new_values, last_sum):
                        return sum(new_values) + (last_sum or 0) 
                
                    lines = ssc.socketTextStream(sys.argv[1], int(sys.argv[2]))
                    running_counts = lines.flatMap(lambda line: line.split(" "))\
                                          .map(lambda word: (word, 1))\
                                          .updateStateByKey(updateFunc, initialRDD=initialStateRDD) 
                    running_counts.pprint()
                    ssc.start()
                    ssc.awaitTermination()
                ```
                运行nc程序
                ```
                nc -lk 9999
                ```
                运行wordcount
                ```
                # 另起终端
                cd /usr/local/spark/mycode/streaming/
                /usr/local/spark/bin/spark-submit --master local[2] NetworkWordCountStateful.py localhost 9999
                # 在nc的窗口输入字符，wordcount窗口就可以显示统计词频
                ```
- 输出操作
    - DStream输出写到文本
        ```python
        cd /usr/local/spark/mycode/streaming/
        vim NetworkWordCountStatefulText.py
        # 写入代码
        
        from __future__ import print_function
        import sys
        from pyspark import SparkContext
        from pyspark.streaming import StreamingContext
        
        if __name__ == "__main__":
            if len(sys.argv) != 3:
                print("Usage: NetworkWordCountStateful.py <hostname> <port>", file=sys.stderr)
                exit(-1)
            sc = SparkContext(appName="PythonStreamingStatefulNetworkWordCount")
            ssc = StreamingContext(sc, 1)
            ssc.checkpoint("file:///usr/local/spark/mycode/streaming/stateful/")
            # RDD with initial state (key, value) pairs
            initialStateRDD = sc.parallelize([(u'hello', 1), (u'world', 1)])
            def updateFunc(new_values, last_sum):
                return sum(new_values) + (last_sum or 0)
            lines = ssc.socketTextStream(sys.argv[1], int(sys.argv[2]))
            running_counts = lines.flatMap(lambda line: line.split(" "))\
                                  .map(lambda word: (word, 1))\
                                  .updateStateByKey(updateFunc, initialRDD=initialStateRDD)
            running_counts.saveAsTextFiles("file:///usr/local/spark/mycode/streaming/stateful/output")
            running_counts.pprint()
            ssc.start()
            ssc.awaitTermination()

        ```
        运行nc程序
        ```
        nc -lk 9999
        ```
        运行保存文本程序
        ```
        # 另起终端
        cd /usr/local/spark/mycode/streaming/
        /usr/local/spark/bin/spark-submit --master local[2] NetworkWordCountStatefulText.py localhost 9999
        # 在nc的窗口输入字符，文本不断被保存
        ```
    - DStream输出到Mysql
        - [ ] 启动MySQL数据库，并完成数据库和表的创建：
            ```
            service mysql start
            mysql -u root -p
            # mysql下写入
            use spark;
            create table wordcount (word char(20), count int(4));
            ```
        - [ ] python安装pysql依赖
            ```
            pip install -i https://pypi.tuna.tsinghua.edu.cn/simple PyMySQL
            ```
        - [ ] 写入程序
            ```python
            cd /usr/local/spark/mycode/streaming/
            vim NetworkWordCountStatefulDB.py
            # 里面写入
            from __future__ import print_function 
            import sys 
            import pymysql 
            from pyspark import SparkContext
            from pyspark.streaming import StreamingContext 
            
            if __name__ == "__main__":
                if len(sys.argv) != 3:
                    print("Usage: NetworkWordCountStateful <hostname> <port>", file=sys.stderr)
                    exit(-1)
                sc = SparkContext(appName="PythonStreamingStatefulNetworkWordCount")
                ssc = StreamingContext(sc, 1)
                ssc.checkpoint("file:///usr/local/spark/mycode/streaming/stateful") 
                # RDD with initial state (key, value) pairs
                initialStateRDD = sc.parallelize([(u'hello', 1), (u'world', 1)])
                
                def updateFunc(new_values, last_sum):
                    return sum(new_values) + (last_sum or 0) 
            
                lines = ssc.socketTextStream(sys.argv[1], int(sys.argv[2]))
                running_counts = lines.flatMap(lambda line: line.split(" "))\
                                      .map(lambda word: (word, 1))\
                                      .updateStateByKey(updateFunc, initialRDD=initialStateRDD) 
                running_counts.pprint()     
                
                # 连接mysql写入数据
                def dbfunc(records):
                    db = pymysql.connect("localhost","root","1234","spark")
                    cursor = db.cursor() 
                    def doinsert(p):
                        sql = "insert into wordcount(word,count) values ('%s', '%s')" % (str(p[0]), str(p[1]))
                        try:
                            cursor.execute(sql)
                            db.commit()
                        except:
                            db.rollback()
                    for item in records:
                        doinsert(item) 
                
                # rdd重分区，对每一分区都执行写入mysql操作
                def func(rdd):
                    repartitionedRDD = rdd.repartition(1)
                    repartitionedRDD.foreachPartition(dbfunc)
            
                running_counts.foreachRDD(func)
                ssc.start()
                ssc.awaitTermination()

            ```
            运行nc程序
            ```
            nc -lk 9999
            ```
            运行保存文本程序
            ```
            # 另起终端
            cd /usr/local/spark/mycode/streaming/
            /usr/local/spark/bin/spark-submit --master local[2] NetworkWordCountStatefulDB.py localhost 9999
            # 在nc的窗口输入字符，可查看mysql是否有新数据增加
            ```