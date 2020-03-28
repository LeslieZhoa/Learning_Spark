### Structured Streaming
- 特点：<br>
    更快，数据抽象为DataFrame
- 关键思想：
    - [ ] 将实时数据流视为一张不断添加数据的表（无界表）
    - [ ] 流计算等同于在一个静态表批处理查询
    - [ ] 查询结果生成结果表，每个一定时间触发计算更新表
- 模式:
    - [ ] 微批处理：要写入日志，数据写入和数据到达会有超过100ms延迟，数据只被处理一次
    - [ ] 持续处理：异步写入日志，可实现毫秒级，保证数据至少被处理一次
- 步骤:
    - 导入pyspark模块
    - 创建SparkSession对象
    - 创建输入数据源
    - 定义流计算过程
    - 启动流计算并输出结果<br>
    *实例任务:*<br>
        一个包含很多行英文语句的数据流源源不断到达，Structured Streaming程序对每行英文语句进行拆分，并统计每个单词出现的频率
        1. 启动hdfs
            ```
            start-all.sh
            ```
        2. 写入主程序代码
            ```python
            mkdir /usr/local/spark/mycode/structuredstreaming
            cd /usr/local/spark/mycode/structuredstreaming
            vim StructuredNetworkWordCount.py
            # StructuredNetworkWordCount.py 写入以下内容
            # 导入PySpark模块，代码如下：

            from pyspark.sql import SparkSession
            from pyspark.sql.functions import split
            from pyspark.sql.functions import explode
            
            # 创建一个SparkSession对象，代码如下：
            if __name__ == "__main__":
                spark = SparkSession \
                    .builder \
                    .appName("StructuredNetworkWordCount") \
                    .getOrCreate()
                spark.sparkContext.setLogLevel('ERROR') #只输出结果和error信息
                
                # 创建输入数据源
                lines = spark \
                    .readStream \
                    .format("socket") \
                    .option("host", "localhost") \
                    .option("port", 9999) \
                    .load()
                
                # 定义流计算过程
                words = lines.select(
                    explode(
                        split(lines.value, " ")
                    ).alias("word")
                )
                # alias-->命名列名称
                wordCounts = words.groupBy("word").count()
                
                # 启动流计算并输出结果
                query = wordCounts \
                .writeStream \
                .outputMode("complete") \
                .format("console") \
                .trigger(processingTime="8 seconds") \
                .start()
            
                query.awaitTermination()

            ```
        3. 启动nc
            ```
            nc -lk 9999
            # 该终端命名为nc终端
            ```
        4. 运行
            ```
            # 另起终端
            cd /usr/local/spark/mycode/structuredstreaming
            /usr/local/spark/bin/spark-submit --master local[2] StructuredNetworkWordCount.py
            
            # 在nc终端写入字符，在本终端查看结果
            ```
- 接收各种不同源
    - 文件源
        - [ ] 生成源测试数据

            ```python
            cd /usr/local/spark/mycode/structuredstreaming
            vim spark_ss_filesource_generate.py
            
            # spark_ss_filesource_generate.py写入以下内容
            # 导入需要用到的模块
            import os
            import shutil
            import random
            import time
            
            TEST_DATA_TEMP_DIR = '/tmp/'
            TEST_DATA_DIR = '/tmp/testdata/'
            
            ACTION_DEF = ['login', 'logout', 'purchase']
            DISTRICT_DEF = ['fujian', 'beijing', 'shanghai', 'guangzhou']
            JSON_LINE_PATTERN = '{{"eventTime": {}, "action": "{}", "district": "{}"}}\n'
            
            # 测试的环境搭建，判断文件夹是否存在，如果存在则删除旧数据，并建立文件夹
            def test_setUp():
                if os.path.exists(TEST_DATA_DIR):
                    shutil.rmtree(TEST_DATA_DIR, ignore_errors=True)
                os.mkdir(TEST_DATA_DIR)
                
            # 测试环境的恢复，对文件夹进行清理
            def test_tearDown():
                if os.path.exists(TEST_DATA_DIR):
                    shutil.rmtree(TEST_DATA_DIR, ignore_errors=True)
            
            # 生成测试文件
            def write_and_move(filename, data):
                with open(TEST_DATA_TEMP_DIR + filename,
                          "wt", encoding="utf-8") as f:
                    f.write(data)
            
                shutil.move(TEST_DATA_TEMP_DIR + filename,
                            TEST_DATA_DIR + filename)
                            
            
            
            if __name__ == "__main__":
                test_setUp()
            
                for i in range(1000):
                    filename = 'e-mall-{}.json'.format(i)
            
                    content = ''
                    rndcount = list(range(100))
                    random.shuffle(rndcount)
                    for _ in rndcount:
                        content += JSON_LINE_PATTERN.format(
                            str(int(time.time())),
                            random.choice(ACTION_DEF),
                            random.choice(DISTRICT_DEF))
                    write_and_move(filename, content)
            
                    time.sleep(1)
            
                test_tearDown()
            ```
        - [ ] 创建程序对数据进行统计

            ```
            cd /usr/local/spark/mycode/structuredstreaming
            vim spark_ss_filesource.py
            # spark_ss_filesource.py 写入以下内容
            # 导入需要用到的模块
            import os
            import shutil
            from pprint import pprint
            
            from pyspark.sql import SparkSession
            from pyspark.sql.functions import window, asc
            from pyspark.sql.types import StructType, StructField
            from pyspark.sql.types import TimestampType, StringType
            # 定义JSON文件的路径常量
            TEST_DATA_DIR_SPARK = 'file:///tmp/testdata/'
            
            if __name__ == "__main__":
                # 定义模式，为时间戳类型的eventTime、字符串类型的操作和省份组成
                schema = StructType([
                    StructField("eventTime", TimestampType(), True),
                    StructField("action", StringType(), True),
                    StructField("district", StringType(), True)])
            
                spark = SparkSession \
                    .builder \
                    .appName("StructuredEMallPurchaseCount") \
                    .getOrCreate()
            
                spark.sparkContext.setLogLevel('ERROR')
                
                lines = spark \
                    .readStream \
                    .format("json") \
                    .schema(schema) \
                    .option("maxFilesPerTrigger", 100) \
                    .load(TEST_DATA_DIR_SPARK)
            
                # 定义窗口
                windowDuration = '1 minutes'
            
                windowedCounts = lines \
                    .filter("action = 'purchase'") \
                    .groupBy('district', window('eventTime', windowDuration)) \
                    .count() \
                    .sort(asc('window'))
                    
                query = windowedCounts \
                    .writeStream \
                    .outputMode("complete") \
                    .format("console") \
                    .option('truncate', 'false') \
                    .trigger(processingTime="10 seconds") \
                    .start()
            
                # option('truncate', 'false')-->不截断全部要打印
                query.awaitTermination()
            ```
        - [ ] 运行
            ```
            # 启动hdfs
            # 新建一个终端生成数据
            cd /usr/local/spark/mycode/structuredstreaming
            python spark_ss_filesource_generate.py
            
            # 新建终端统计数据
            cd /usr/local/spark/mycode/structuredstreaming
            /usr/local/spark/bin/spark-submit --master local[2] spark_ss_filesource.py
            ```
    - kafka
        - [ ] 准备
            ```
            # 开启zookeeper,三个服务器都运行
            cd /usr/local/kafka
            ./bin/zookeeper-server-start.sh config/zookeeper.properties
            # 窗口不要关，另起其他端口
            
            # 开启kafka，三个服务器都运行
            cd /usr/local/kafka
            bin/kafka-server-start.sh config/server.properties
            # 窗口不要关，另起其他端口
            ```
        - [ ] 监控输入文本流
            ```
            cd /usr/local/kafka
            bin/kafka-console-consumer.sh --bootstrap-server 192.168.1.128:9092,192.168.1.129:9092,192.168.1.130:9092  --topic wordcount-topic
            ```
        - [ ] 监控输出文本流
            ```
            cd /usr/local/kafka
            bin/kafka-console-consumer.sh --bootstrap-server 192.168.1.128:9092,192.168.1.129:9092,192.168.1.130:9092 --topic wordcount-result-topic
            ```
        - [ ] 编写producer生产者
            ```
            cd /usr/local/spark/mycode/structuredstreaming
            vim spark_ss_kafka_producer.py
            # spark_ss_kafka_producer.py下写如下内容
            import string
            import random
            import time
            
            from kafka import KafkaProducer 
            if __name__ == "__main__":
                producer = KafkaProducer(bootstrap_servers=['192.168.1.128:9092','192.168.1.129:9092','192.168.1.130:9092'])
            
                while True:
                    s2 = (random.choice(string.ascii_lowercase) for _ in range(2))
                    word = ''.join(s2)
                    value = bytearray(word, 'utf-8')
            
                    producer.send('wordcount-topic', value=value) \
                        .get(timeout=10)
            
                    time.sleep(0.1)
            
            ```
        - [ ] 安装kafka python环境
            ```
            pip install  kafka-python
            ```
        - [ ] 运行生产者
            ```
            cd /usr/local/spark/mycode/structuredstreaming/
            python spark_ss_kafka_producer.py
            
            # 可在输入监视终端查看
            ```
        - [ ] 编写消费者程序
            ```python
            cd /usr/local/spark/mycode/structuredstreaming/
            vim spark_ss_kafka_consumer.py
            # spark_ss_kafka_consumer.py写入如下
            from pyspark.sql import SparkSession
            
            
            if __name__ == "__main__":
                spark = SparkSession \
                    .builder \
                    .appName("StructuredKafkaWordCount") \
                    .getOrCreate()
            
                spark.sparkContext.setLogLevel('ERROR')
                
                lines = spark \
                    .readStream \
                    .format("kafka") \
                    .option("kafka.bootstrap.servers", "192.168.1.128:9092,192.168.1.129:9092,192.168.1.130:9092") \
                    .option("subscribe", 'wordcount-topic') \
                    .load() \
                    .selectExpr("CAST(value AS STRING)")
            
                wordCounts = lines.groupBy("value").count()
                
                query = wordCounts \
                    .selectExpr("CAST(value AS STRING) as key", "CONCAT(CAST(value AS STRING), ':', CAST(count AS STRING)) as value") \
                    .writeStream \
                    .outputMode("complete") \
                    .format("kafka") \
                    .option("kafka.bootstrap.servers", "192.168.1.128:9092,192.168.1.129:9092,192.168.1.130:9092") \
                    .option("topic", "wordcount-result-topic") \
                    .option("checkpointLocation", "file:///tmp/kafka-sink-cp") \
                    .trigger(processingTime="8 seconds") \
                    .start()
            
                query.awaitTermination()
                
            ```
        - [ ] 运行消费程序
            ```
            cd /usr/local/spark/mycode/structuredstreaming
            /usr/local/spark/bin/spark-submit --master local[2]  --packages org.apache.spark:spark-sql-kafka-0-10_2.11:2.4.5 spark_ss_kafka_consumer.py
            # 可在消费终端查看统计结果
            ```
    - rate源:<br>
        Rate源可每秒生成特定个数的数据行，每个数据行包括时间戳和值字段。时间戳是消息发送的时间，值是从开始到当前消息发送的总个数，从0开始。Rate源一般用来作为调试或性能基准测试。
        
        ```
        cd /usr/local/spark/mycode/structuredstreaming
        vim spark_ss_rate.py
        # 在spark_ss_rate.py写入
        from pyspark.sql import SparkSession
        
        
        if __name__ == "__main__":
            spark = SparkSession \
                .builder \
                .appName("TestRateStreamSource") \
                .getOrCreate()
        
            spark.sparkContext.setLogLevel('ERROR')
            
            lines = spark \
            .readStream \
            .format("rate") \
            .option('rowsPerSecond', 5) \
            .load()
        
            print(lines.schema)
        
            query = lines \
                .writeStream \
                .outputMode("update") \
                .format("console") \
                .option('truncate', 'false') \
                .start()
        
            query.awaitTermination()
        
        
        ```
        运行
        ```
        cd /usr/local/spark/mycode/structuredstreaming
        /usr/local/spark/bin/spark-submit   --master local[2] spark_ss_rate.py
        ```
- 输出操作<br>
    DataFrame/Dataset的.writeStream()方法将会返回DataStreamWriter接口，接口通过start()真正启动流计算，并将DataFrame/Dataset写入到外部的输出接收器
    - DataStream/Writer接口函数
        - [ ] format:接收器类型：
            1. File接收器
            2. Kafka接收器
            3. Foreach接收器
            4. Console接收器
            5. Memory接收器<br>
            *Console接收器和Memory接收器仅用于调试，有些接收器无法保证输出持久性，导致其不容错*
        - [ ] outputMode：输出模式，指定写入接收器内容
            1. Append模式：只有结果表中自上次触发间隔后增加的新行，才会被写入外部存储器。这种模式一般适用于“不希望更改结果表中现有行的内容”的使用场景。
            2. Complete模式：已更新的完整的结果表可被写入外部存储器
            3. Update模式：只有自上次触发间隔后结果表中发生更新的行，才会被写入外部存储器。这种模式与Complete模式相比，输出较少，如果结果表的部分行没有更新，则不会输出任何内容。当查询不包括聚合时，这个模式等同于Append模式。
        - [ ] queryName：查询名称，可选，用于标识查询的唯一名称
        - [ ] trigger：触发间隔，可选，如未指定，系统将上一次处理完成后立即检查新数据可用性。若先前处理尚未完成导致超过触发间隔，系统将在处理完成后立即触发新查询
    
