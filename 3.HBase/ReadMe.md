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