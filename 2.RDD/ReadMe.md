### RDD
1. #### RDD工作原理<br>
    - RDD本质上是一个只读的分区记录集合，加载高度受限共享内存模型即生成在内存当中的内存集合，且不能修改。RDD转换操作是创建新的RDD数据，在转换过程中进行修改
    - 操作类型：<br>
        粗粒度转换，一次只能针对RDD全集转换
        - 转换类型操作（Transformation）：只记录转换轨迹，不发生计算
        - 动作类型操作（Action）:动作时触发计算
        - 小例子说明：当作转换时若发生错误，并不报错，只有动作操作时有错误才报错
    - 数据丢失怎么办：找血亲关系，寻亲恢复数据-->儿子找爸爸；中间结果持久化到内存中
    - 依赖关系：
        并行执行任务框架-->fork/join机制，可看作fork就是干活，join就是等人做继续干活准备
        - 窄依赖：
            1. 一个父亲对一个儿子或多个父亲对一个儿子；
            2. 无shuffle，也就是不用join等人，每个人完成自己任务互不干扰；
            3. 可进行流水线优化（剔除没必要的join）；
            4. 无需等待所有数据可流水线
        - 宽依赖：
            1. 一个父亲对应多个儿子；
            2. 有shuffle，就是等着所有人完成一项任务之后再重新打乱分配任务，如果不参见join等人就进行不下去；
            3. 无法进行流水线优化，因为只要shuffle就一定会写磁盘要等所有数据都转换完；
            4. 不可流水线，但可生成不同阶段
2. #### RDD创建
    前提是启动hadoop和spark<br>

    ```  
    # 按安装环境来承接应该启动hadoop和spark的操作为
    start-all.sh
    cd /usr/local/spark
    sbin/start-all.sh
    # 启动pyspark
    bin/pyspark  --master  spark://node-1:7077 
    ```
    #### *血与泪的教训，master地址和读取文件名操作一定要写对啊！！！*
    - 数据加载：
        - 文件系统中加载：
            1. 读取本地系统
                ```
                # 如果分布式每个服务器都应有该文件，而且目录内容一致,以下代码在pyspark内运行
                lines = sc.textFile("file:///usr/local/spark/mycode/rdd/word.txt")
                # 本地文件格式file:///+相应路径
                lines.collect() # 看看里面有啥
                
                ```
            2. 读取分布式文件系统HDFS
                ```
                # 首先得把文件放到hdfs里，在shell运行
                hadoop fs -put /usr/local/spark/mycode/rdd/word.txt /
                
                # 读取文件，以下代码在pyspark内运行
                lines = sc.textFile("/word.txt")
                lines.collect() # 看看里面有啥
                ```
            3. 读取云端数据
        - 通过并行集合数组创建RDD
            ```
            # 在pyspark运行
            array = [1,2,3,4,5]
            rdd = sc.parallelize(array)
            rdd.collect()

            ```
    - RDD转换操作：<br>
        每一次转换成新的RDD，并生成DAG图，以下代码均在pyspark上运行
        - filter(func)-->筛选出满足func的元素
            ```python
            # 假设word.txt里面内容是
            # Hadoop is good
            # Spark is better
            # Spark is fast
            lines = sc.textFile("file:///usr/local/spark/mycode/rdd/word.txt")
            linesWithSpark = lines.filter(lambda line: "Spark" in line)
            linesWithSpark.collect()
            # 得到的结果是字符串中包含Spark的
            # ['Spark is better','Spark is fast']
            ```
        - map(func)--> 把每个元素传入func得到新的元素
            ```python
            data = [1,2,3,4,5]
            rdd1 = sc.parallelize(data)
            rdd2 = rdd1.map(lambda x:x+10)
            rdd2.collect()
            # 得到的是[11,12,13,14,15]

            ```
        - flatMap(func)-->把元素按照map(func)得到之后再平铺出来
            ```python
            lines = sc.textFile("file:///usr/local/spark/mycode/rdd/word.txt")
            words = lines.flatMap(lambda line:line.split(" "))
            words.collect()
            # 得到的结果为
            # ['Hadoop', 'is', 'good','Spark', 'is', 'fast','Spark', 'is', 'better']
            
            ```
        - groupByKey-->将相同键值的value组成一个迭代器
            ```python
            words = sc.parallelize([("Hadoop",1),("is",1),("good",1), \
            ("Spark",1),("is",1),("fast",1),("Spark",1),("is",1),("better",1)])
            words1 = words.groupByKey()
            words1.collect()
            # 结果自己运行看吧
            ```
        - reduceByKey(func)-->经过groupByKey，再把相同键值迭代器里的元素进行func操作
            ```python
            words = sc.parallelize([("Hadoop",1),("is",1),("good",1),("Spark",1), \
            ("is",1),("fast",1),("Spark",1),("is",1),("better",1)])
            # 做相同键值value求和
            words1 = words.reduceByKey(lambda a,b:a+b)
            words1.collect()
            # 得到的结果为：
            # [('good', 1),('Hadoop', 1),('better', 1),('Spark', 2),('fast', 1),('is', 3)]
            ```
    - RDD行动操作
        - count()-->返回元素个数
        - collect()-->以数组形式返回数据集中所有元素
        - first()-->返回第一个元素
        - take(n)-->以数组形式返回数据集中前n个元素
        - reduce(func)-->通过func(通过两个参数返回一个值)聚合数据集中的元素
        - foreach(func)-->将数据集中每个元素传递func运行(若多集群func为print不打印)
    - 持久化：
        - 形式：
            1. .persist()-->对一个RDD标记为持久化，当动作操作才会触发
            2. .persist(MEMORY_ONLY)==.catch()-->只存内存中
            3. .persist(MEMORY_AND_DISK)-->内存存不下放磁盘里
            4. .unpersist()-->手动释放
        - 例子
            ```python
            list = ["Hadoop","Spark","Hive"]
            rdd = sc.parallelize(list)
            rdd.cache()  #会调用persist(MEMORY_ONLY)，但是，语句执行到这里，并不会缓存rdd，因为这时rdd还没有被计算生成
            print(rdd.count()) #第一次行动操作，触发一次真正从头到尾的计算，这时上面的rdd.cache()才会被执行，把这个rdd放到缓存中
            # 输出为3
             print(','.join(rdd.collect())) #第二次行动操作，不需要触发从头到尾的计算，只需要重复使用上面缓存中的rdd
            # 输出为Hadoop,Spark,Hive

            ```
    - 分区
        - 优点：增加并行度，减少通信开销
        - 各种模式spark.default.parallelism
            1. local模式-->默认本地CPU数量，若设置local[N],则为N
            2. Apache Mesos 默认8
            3. standalone，YARN模式，默认max(cpu总数,2)
        - 小例子
            ```python
            # 第一种形式
            lines = sc.textFile("file:///usr/local/spark/mycode/rdd/word.txt",2)
            # 第二种形式
            data = sc.parallelize([1,2,3,4,5],2)
            len(data.glom().collect())  #显示data这个RDD的分区数量
            # 应该输出2
            rdd = data.repartition(1)   #对data这个RDD进行重新分区
            len(rdd.glom().collect())   #显示rdd这个RDD的分区数量
            # 应该输出1

            ```
        - 分区方式：哈希分区，自定义分区（.partitionBy(number,func)-->对应分区数和分区函数，只接受键值对），区域分区
        - 大例子
        
        ```python
        # 在/usr/local/spark/mycode/python/目录下新建一个TestPartitioner.py文件并写入一下内容
        
        # 功能是根据key值的最后一位数字，写到不同的文件
        from pyspark import SparkConf, SparkContext

        def MyPartitioner(key):
            print("MyPartitioner is running")
            print('The key is %d' % key)
            return key%10

        def main():
            print("The main function is running")
            conf  =  SparkConf().setMaster("local").setAppName("MyApp")
            sc  =  SparkContext(conf = conf)
            data = sc.parallelize(range(10),5)
            data.map(lambda x:(x,1)) \
                   .partitionBy(10,MyPartitioner) \
                   .map(lambda x:x[0]) \
                   .saveAsTextFile("file:///usr/local/spark/mycode/rdd/partitioner")

        if __name__ == '__main__':
               main()

        ```
        
        写完之后就得运行了(在命令行运行)
        ```
        cd /usr/local/spark/mycode/python/
        python TestPartitioner.py
        ```
    - 键值对转换
        - reduceByKey(func)
        - groupByKey()
        - keys
        - values
        - combineByKey
        - join-->保留相同key，value合并在一起
            ```python
            # 在pyspark运行
            pairRDD1 = sc. \
            parallelize([("spark",1),("spark",2),("hadoop",3),("hadoop",5)])
            pairRDD2 = sc.parallelize([("spark","fast")])
            pairRDD3 = pairRDD1.join(pairRDD2)
            pairRDD3.collect()
            # 最后输出结果[('spark', (1, 'fast')),('spark', (2, 'fast'))]
            ```
        - mapValues(func)-->key不变，映射新值
            ```python
            # 在pyspark运行
            list = [("Hadoop",1),("Spark",1),("Hive",1),("Spark",1)]
            pairRDD = sc.parallelize(list)
            pairRDD1 = pairRDD.mapValues(lambda x:x+1)
            pairRDD1.collect()
            # 输出[('Hadoop', 2),('Spark', 2),('Hive', 2),('Spark', 2)]
 
            ```
        - sortByKey()
        - sortBy(func)
        - 一个综合例子
            ```python
            # 在pyspark运行,功能是：
            # 键值对的key表示图书名称，value表示某天图书销量
            # 请计算每个键对应的平均值，也就是计算每种图书的每天平均销量。
            rdd = sc.parallelize([("spark",2),("hadoop",6),("hadoop",4),("spark",6)])
            rdd.mapValues(lambda x:(x,1)).\
            reduceByKey(lambda x,y:(x[0]+y[0],x[1]+y[1])).\
            mapValues(lambda x:x[0]/x[1]).collect()
            # 应该输出[('hadoop', 5.0), ('spark', 4.0)]

            ```
    - 文件数据读取
        ```python
        # 文件读取
        textFile = sc.\
        textFile("file:///usr/local/spark/mycode/rdd/word.txt")
        # 文件存储，指定文件夹，因为可能因为分区生成不止一个文件
        # 存储本地文件
        textFile.\
        saveAsTextFile("file:///usr/local/spark/mycode/rdd/writeback")
        # 存储hdfs文件
        textFile.saveAsTextFile("writeback")

        ```
            