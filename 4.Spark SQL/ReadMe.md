### Spark SQL
- 特点：
    - [ ] 带有模式信息的RDD
    - [ ] 支持多种数据转换成DataFrame
- 优势：
    - [ ] 提供DataFrame API，可对内部外部各种数据源执行各种关系操作
    - [ ] 支持大量数据源和数据分析算法
    - [ ] 可以融合，传统关系数据库结构化数据管理能力和机器学习算法数据处理能力
- DataFrame与RDD区别
    - RDD是分布式java对象集合，对象内部结构对RDD而言不可知
    - DataFrame，是一种以RDD为基础分布式数据集，提供详细结构信息
    - 举例子：假设数据为小明，年龄，性别等信息；RDD只能看到相当于只能看到标有小明的袋子，其他信息要打开袋子才能看到；DataFrame就能直接清晰看到所有信息
- 接口说明
    - SparkContext--> sc是RDD执行官
    - SparkSession--> spark是DataFrame执行官
- 创建DataFrame
    - [ ] 运行各种环境
        ```
        # 运行hadoop
        start-all.sh
        # 运行spark
        cd /usr/local/spark
        sbin/spark-all.sh
        ```
    - [ ] 读取dataframe
        ```
        # 以下代码都以在pyspark下运行为例
        spark.read.text("people.txt") # 读取文本文件people.txt创建DataFrame
        spark.read.json("people.json") #读取people.json文件创建DataFrame；在读取本地文件或HDFS文件时，要注意给出正确的文件路径
        spark.read.parquet(“people.parquet”) # 读取people.parquet文件创建DataFrame
        
        # 或者也可以使用如下格式的语句：
        spark.read.format("text").load("people.txt") #读取文本文件people.json创建DataFrame；
        spark.read.format("json").load("people.json") #读取JSON文件people.json创建DataFrame；
        spark.read.format("parquet").load("people.parquet") #读取Parquet文件people.parquet创建DataFrame。

        ```
        一个实例
        ```
        # 运行pyspark
        bin/pyspark
        
        # 在pyspark运行下列代码
        df = spark.read.json("file:///usr/local/spark/examples/src/main/resources/people.json")

        df.show()

        ```
    - [ ] 写入DataFrame
        ```
        df.write.text("people.txt")
        df.write.json("people.json")
        df.write.parquet("people.parquet")
        
        # 或者
        df.write.format("text").save("people.txt")
        df.write.format("json").save("people.json")
        df.write.format ("parquet").save("people.parquet")

        ```
        一个实例
        ```
        peopleDF = spark.read.format("json").\
        ...load("file:///usr/local/spark/examples/src/main/resources/people.json")
        peopleDF.select("name", "age").write.format("json").\
        ...save("file:///usr/local/spark/mycode/sparksql/newpeople.json")
        peopleDF.select("name").write.format("text").\
        ...save("file:///usr/local/spark/mycode/sparksql/newpeople.txt")
        ```
    - 常用操作
        ```
        df = peopleDF
        # 打印模式信息
        df.printSchema()
        
        # 查询
        df.select(df["name"],df["age"]+1).show()
        
        # 过滤信息
        df.filter(df["age"]>20).show()
        
        # 分组
        df.groupBy("age").count().show()
        
        # 排序 desc降序排序，asc升序排序
        df.sort(df["age"].desc()).show()
        
        # 两个指标排序
        df.sort(df["age"].desc(),df["name"].asc()).show()
        ```
- 从RDD转换得到DataFrame
    - 反射机制推断RDD模式<br>
        可提前获知数据结构
        ```
        # people.txt，其内容如下：
        Michael, 29
        Andy, 30
        Justin, 19
        
        # 功能 现在要把people.txt加载到内存中生成一个DataFrame，并查询其中的数据

        # 运行pyspark
        bin/pyspark
        
        # 在pyspark运行下列代码
        
        # ROW封装一行数据
        from pyspark.sql import Row
        
        # 返回一个sparkContext对象
        people = spark.sparkContext.\
        ... textFile("file:///usr/local/spark/examples/src/main/resources/people.txt").\
        ... map(lambda line: line.split(",")).\
        ... map(lambda p: Row(name=p[0], age=int(p[1])))
        
        # 创建DataFrame
        schemaPeople = spark.createDataFrame(people)
        
        #必须注册为临时表才能供下面的查询使用
        schemaPeople.createOrReplaceTempView("people") 
        personsDF = spark.sql("select name,age from people where age > 20")
        
        #DataFrame中的每个元素都是一行记录，包含name和age两个字段，分别用p.name和p.age来获取值
        personsRDD=personsDF.rdd.map(lambda p:"Name: "+p.name+ ","+"Age: "+str(p.age))
        
        personsRDD.collect()
        
        ```
    - 编程方式定义RDD模式<br>
        当无法提前获知数据结构时，就需要采用编程方式定义RDD模式
        ```
        # 功能：现在需要通过编程方式把people.txt加载进来生成DataFrame，并完成SQL查询。
        
        from pyspark.sql.types import *
        from pyspark.sql import Row
        
        #下面生成“表头”
        schemaString = "name age"
        fields = [StructField(field_name, StringType(), True) for field_name in schemaString.split(" ")]
        schema = StructType(fields)
        
        #下面生成“表中的记录”
        lines = spark.sparkContext.\
        ... textFile("file:///usr/local/spark/examples/src/main/resources/people.txt")
        parts = lines.map(lambda x: x.split(","))
        people = parts.map(lambda p: Row(p[0], p[1].strip()))
        
        #下面把“表头”和“表中的记录”拼装在一起
        schemaPeople = spark.createDataFrame(people, schema)
        
        #注册一个临时表供下面查询使用
        schemaPeople.createOrReplaceTempView("people")
        results = spark.sql("SELECT name,age FROM people")
        results.show()

        ```
- 安装mysql
    - 换源(为了速度)
        ```
        cd /etc/apt
        # 备份
        sudo cp sources.list sources.list.bak
        
        # 修改
        sudo vi sources.list 
        # 将文件里内容删除替换成
        deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted

        deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted
        
        deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial universe
        
        deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates universe
        
        deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial multiverse
        
        deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates multiverse
        
        deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
        
        deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted
        
        deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security universe deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security multiverse
        
        # 更新apt
        sudo apt-get update
        
        ```
    - 安装mysql
        ```
        # 安装过程中需设置root密码
        sudo apt-get update  #更新软件源
        sudo apt-get install mysql-server  #安装mysql
        
        # 如出现依赖库错误运行下面代码
        sudo apt-get -f install
        sudo apt-get install mysql-server  #安装mysql
        
        # 设置utf-8
        sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
        # 在lc-messages-dir = /usr/share/mysql下面添加
        character_set_server=utf8
        
        # 重启mysql
        service mysql restart
        
        # 进入mysql 输入密码
        mysql -u root -p
        
        # 在sql里运行
        show variables like "char%";
        
        # 若character_set_server为utf-8则成功
        
        
        ```
    
    - 创建spark表
        ```
        # 进入mysql 输入密码
        mysql -u root -p
        
        # 在sql里运行
        create database spark;
        use spark;
        create table student (id int(4), name char(20), gender char(4), age int(4));
        insert into student values(1,'Xueqian','F',23);
        insert into student values(2,'Weiliang','M',24);
        select * from student
        ```
    - 配置JBDC<br>
        - [ ] 点击[此链接](https://dev.mysql.com/downloads/connector/j/)进入jbdc下载页面
        - [ ] Select Operating System选择Platform Independent，下载TAR Archive那个，并解压
        - [ ] 将解压后里面的.jar文件放入/usr/local/spark/jars/下，我的文件名为mysql-connector-java-8.0.19.jar
- pyspark读写mysql
    - 写入mysql<br>
        /usr/local/spark/mycode/sparksql/下创建WriteSql.py文件
        ```
        在WriteSql.py下写入
        from pyspark.sql import Row
        from pyspark.sql.types import *
        from pyspark import SparkContext,SparkConf
        from pyspark.sql import SparkSession
        
        spark = SparkSession.builder.config(conf = SparkConf()).getOrCreate()
        
        #下面设置模式信息
        schema = StructType([StructField("id", IntegerType(), True), \
        StructField("name", StringType(), True), \
        StructField("gender", StringType(), True), \
        StructField("age", IntegerType(), True)])
        #下面设置两条数据，表示两个学生的信息
        studentRDD = spark \
        .sparkContext \
        .parallelize(["3 Rongcheng M 26","4 Guanhua M 27"]) \
        .map(lambda x:x.split(" "))
         
        #下面创建Row对象，每个Row对象都是rowRDD中的一行
        rowRDD = studentRDD.map(lambda p:Row(int(p[0].strip()), p[1].strip(), p[2].strip(), int(p[3].strip())))
         
        #建立起Row对象和模式之间的对应关系，也就是把数据和模式对应起来
        studentDF = spark.createDataFrame(rowRDD, schema)
         
        #写入数据库
        prop = {}
        prop['user'] = 'root'
        prop['password'] = '1234'
        prop['driver'] = "com.mysql.jdbc.Driver"
        studentDF.write.jdbc("jdbc:mysql://localhost:3306/spark",'student','append', prop)

        ```
        运行代码
        ```
        cd /usr/local/spark
        bin/spark-submit --jars /usr/local/spark/jars/mysql-connector-java-8.0.19.jar /usr/local/spark/mycode/sparksql/WriteSql.py
        
        # 进入mysql查询student表是否插入数据
        ```
    - 读取mysql<br>
        运行pyspark
        ```
        cd /usr/local/spark
        ./bin/pyspark
        ```
        pyspark运行下列代码
        ```
        jdbcDF = spark.read.format("jdbc").option("driver","com.mysql.jdbc.Driver").option("url", "jdbc:mysql://localhost:3306/spark").option("dbtable", "student").option("user", "root").option("password", "1234").load()
        jdbcDF.show()
        ```
