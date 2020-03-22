## 配置spark集群

1. #### 安装虚拟机和ubuntu环境
    - 点击[此链接](https://download.virtualbox.org/virtualbox/5.2.14/VirtualBox-5.2.14-123301-Win.exe)下载VirtualBox虚拟机，按照提示正常安装。
    - 打开VirtualBox软件，点击管理的全局设定；点击常规，更改默认虚拟电脑位置即把虚拟环境放在哪。具体图示如下：<br>
    <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/1.png?raw=true"/><br>
 <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/2.png?raw=true"/><br>
    - 新建虚拟环境，点击新建，输入环境名称，类型和版本，按下图填写即可；点击下一步，内存大小我选取的是1500(本机运行内存为8gb)，点击下一步；现在创建虚拟硬盘，点击创建；默认VDI模式下一步；默认动态分配，下一步；文件大小选定20GB，点击创建<br>
    <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/3.png?raw=true"/><br>
 <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/4.png?raw=true"/><br>
<img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/5.png?raw=true"/><br>
 <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/6.png?raw=true"/><br>
<img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/7.png?raw=true"/><br>
 <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/8.png?raw=true"/><br>
    - 点击[此链接](http://ftp.ubuntu-tw.org/mirror/ubuntu-releases/16.04.6/ubuntu-16.04.6-desktop-amd64.iso)下载ubuntu1604镜像放置到下载目录中；双击node-1，选择下载的镜像作为启动盘启动。<br>
    <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/9.png?raw=true"/><br>
     <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/10.png?raw=true"/><br>
    - 正常ubuntu安装步骤，重启环境若迟迟不进来，就关掉再重启<br>
    <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/11.png?raw=true"/><br>
     <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/12.png?raw=true"/><br>
<img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/13.png?raw=true"/><br>
     <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/14.png?raw=true"/><br>
    - 安装增强功能，为了方便跨机复制，如果失败就点击如下图的弹出，重新安装<br>
    <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/15.png?raw=true"/><br>
     <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/16.png?raw=true"/><br>
    - 设置可跨机粘贴，点击node-1设置的常规-->高级，共享粘贴版和拖放都选择双向<br>
        <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/22.png?raw=true"/><br>
    - 将用户设置为root.打开终端
        ```shell
        sudo passwd root
        # 输入搭建环境时设置的密码，并设置新的密码
        # 编辑 /etc/lightdm/lightdm.conf 文件
    
        sudo gedit  /etc/lightdm/lightdm.conf
        # 更改为如下：
        [Seat:*]
        autologin-guest=false
        autologin-user=root
        autologin-user-timeout=0
        greeter-session=lightdm-gtk-greeter 
        
        # 编辑/root/.profile文件，增加tty -s &&：
    
        sudo gedit /root/.profile 
    
        # ~/.profile: executed by Bourne-compatible login shells.
        
        if [ "$BASH" ]; then
          if [ -f ~/.bashrc ]; then
            . ~/.bashrc
          fi
        fi
        
        tty -s && mesg n || true 
        
     
        # 重启系统即可 
        ```
    
2. #### 配置静态ip
    
    - 打开windows的cmd输入ipconfig，找到本机的ip地址，例如我的是192.168.199.122，我设置相应虚拟机的ip，node-1:192.168.199.128；node-2:192.168.199.129；node-3:192.168.199.130。<br>
    <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/17.png?raw=true"/><br>
    - 右键node-1点击设置；点击网络选项，连接方式，桥接网卡。<br>
    <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/18.png?raw=true"/><br>
<img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/19.png?raw=true"/><br>
    - 启动node-1服务器，点击右上角的网络连接编辑连接；点击有线连接1，编辑；ipv4设置，方法选手动，增加地址，即上文确定的node-1静态ip，DNS服务器我选择与网关保持一致;保存，继续点击右上角网络连接，点击有线连接1;打开终端输入ifconfig会发现ip改变了<br>
    <img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/20.png?raw=true"/><br>
<img src="https://github.com/LeslieZhoa/Learning_Spark/blob/master/1.%E5%AE%89%E8%A3%85%E7%8E%AF%E5%A2%83/img/21.png?raw=true"/><br>
    - 配置虚拟机到本机映射（三台虚拟机）.进入本机目录，打开hosts文件：C:\Windows\System32\drivers\etc,添加虚拟机IP：
        ```
        192.168.199.128    node-1
        192.168.199.129    node-2
        192.168.199.130    node-3
        ```
        检查是否可以ping node-1
    - 配置三台虚拟机之间的IP映射
        ```
        sudo apt-get update
        sudo apt-get install vim
        vim /etc/hosts
        
        # 注释127.0.1.1      node-1，并在文件末尾添加如下
        192.168.199.128    node-1
        192.168.199.129    node-2
        192.168.199.130    node-3
        ```
    
    - ssh 免密登陆，下载ssh，  sudo apt-get install openssh-server
        ```
        #对本机免密码登录：
        ssh-keygen -t rsa
        cd /root/.ssh
        cp id_rsa.pub authorized_keys
        
       
        ```
        ```
        # 修改ssh，使本地也可ssh到node-1
        vim /etc/ssh/sshd_config 
        # 找到PermitRootLogin prohibit-password将该行改为PermitRootLogin yes，注意去掉前面的# 
        # 重启ssh
        service ssh restart
        ```
3. #### 安装python3
    - 新建一个BigData文件夹用于存储下载数据,mkdir ~/BigData
    - 点击[此链接](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-4.2.0-Linux-x86_64.sh)下载Anaconda-3-4.2.0到BigData
    - 安装Anaconda，运行bash ~/BigData/Anaconda3-4.2.0-Linux-x86_64.sh，其余操作按指示来会自动把Path加入环境中
    - 开启新的终端pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pyspark
3. #### jdk安装（版本一定要是java8）
    - 进入到[java8下载界面](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)下载Linux x64 Compressed Archive到node-1的BigData中（可能需要注册账号才能下载）。
        ```
        # 创建目录
        sudo mkdir /usr/lib/jvm/
        # 解压
        tar zxvf ~/BigData/jdk-8u241-linux-x64.tar.gz -C /usr/lib/jvm/
        mv /usr/lib/jvm/jdk1.8.0_241/ /usr/lib/jvm/java
        # 修改环境变量
        sudo vim ~/.bashrc
        # 末尾添加
        export JAVA_HOME=/usr/lib/jvm/java/  ## 这里要注意目录要换成自己解压的jdk 目录
        export JRE_HOME=${JAVA_HOME}/jre  
        export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib export PATH=${JAVA_HOME}/bin:$PATH  
        
        #  使环境变量马上生效

        source ~/.bashrc
        # 设置系统默认jdk 版本
        sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java/bin/java 300  
        sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/java/bin/javac 300  
        sudo update-alternatives --install /usr/bin/jar jar /usr/lib/jvm/java/bin/jar 300   
        sudo update-alternatives --install /usr/bin/javap javap /usr/lib/jvm/java/bin/javap 300
        
        # 然后执行:
        sudo update-alternatives --config java
        
        # 测试
        java -version
        ```
4. #### 搭建hadoop集群
    - 点击[此链接](http://archive.apache.org/dist/hadoop/core/hadoop-2.7.3/hadoop-2.7.3.tar.gz)下载hadoop包到node-1的BigData(这里选用版本为2.7.3),解压：
        ```
        sudo tar -zxf ~/BigData/hadoop-2.7.3.tar.gz -C /usr/local    # 解压到/usr/local中
        cd /usr/local/
        sudo mv ./hadoop-2.7.3/ ./hadoop 
        ```
    - 配置文件
        ```
        cd /usr/local/hadoop/etc/hadoop
        # 查看java目录
        which java
        # 我的显示 /usr/lib/jvm/java//bin/java
        # 配置java环境
        vim hadoop-env.sh
        # 找到export JAVA_HOME=
        #更改为
        export JAVA_HOME=/usr/lib/jvm/java/
        
        # 配置core-site.xml
        vim core-site.xml
        # 更改<configuration>部分
        <configuration>
            <property>
                <name>fs.defaultFS</name>
                <value>hdfs://node-1:9000</value>
            </property>
        </configuration>
        
        
        # 配置hdfs-site.xml
        vim hdfs-site.xml 
        # 更改<configuration>部分
         <configuration>
            <property>
                <name>dfs.namenode.name.dir</name>
                <value>/usr/local/hadoop/name</value>
            </property>
            <property>
                <name>dfs.datanode.data.dir</name>
                <value>/usr/local/hadoop/data</value>
            </property>
             <property>
                <name>dfs.tmp.dir</name>
                <value>/usr/local/hadoop/tmp</value>
             </property>
            <property>
                <name>dfs.replication</name>
                <value>3</value>
            </property>
        </configuration>
        
        # 配置mapred-site.xml
        mv mapred-site.xml.template mapred-site.xml
        vim mapred-site.xml
        # 更改<configuration>部分
        <configuration>
            <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
            </property>
        </configuration>
        
        # 配置yarn-site.xml
        vim yarn-site.xml
        # 更改<configuration>部分
        <configuration>
            <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>node-1</value>
            </property>
            <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
            </property>
        </configuration>
        
        # 配置slaves
        vim slaves
        #内容删除替换如下：
        node-2
        node-3
        ```
    - 配置环境变量
        ```
        vim /etc/profile
        # 在文件末尾加上
        export HADOOP_HOME=/usr/local/hadoop/
        export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
        export HADOOP_COMMOM_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
        
        # 激活一下
        source /etc/profile
        ```
    
     

5. #### spark集群搭建
    - 点击[spark下载界面](https://spark.apache.org/downloads.html)，我的版本选择是2.4.5，choose a package type是Pre-build with user-provided Apache Hadoop下载spark到node-1的BigData
    - 解压
        ```
        sudo tar -zxf ~/BigData/spark-2.4.5-bin-without-hadoop.tgz -C /usr/local/
        cd /usr/local
        sudo mv ./spark-2.4.5-bin-without-hadoop/ ./spark

        ```
    - 修改文件
        ```
        $ cd /usr/local/spark/conf
        mv spark-env.sh.template spark-env.sh
        mv slaves.template slaves
    
        vim spark-env.sh
        # 在文件最末尾添加
        export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop/bin/hadoop classpath) 
        export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
        # 填写你自己node-1的ip
        export SPARK_MASTER_IP=192.168.199.128


        vim slaves
        # 将内容删除替换如下：
        node-2
        node-3
        
        # 配置python3环境
        vim ~/.bashrc
        # 末尾添加入下
        export SPARK_HOME="/usr/local/spark/"
        # 运行which python得到的地址
        export PYSPARK_PYTHON=/root/anaconda3/bin/python
          
        # 激活环境
        source ~/.bashrc

        ```
    - 复制其他虚拟环境。
        - 退出node-1，右键复制，完全复制node-2和node-3
        - 按照上述方式更改有线连接的ipv4
        - 修改计算机名
            ```
            sudo gedit /etc/hostname
            # 将node-1改为node-2
            # 重启电脑
            ```
        
        - node-3按照同样方法
    - 三台机器之间的免密码登录：
        ```
        # 三台机器都运行
        ssh-copy-id -i node-1
        ssh-copy-id -i node-2
        ssh-copy-id -i node-3
        ```
    - 初始化
        在node-1上运行
        ```
        hadoop namenode -format
        # 启动
        start-dfs.sh
        # 登陆node-1:50070看Live Nodes是否为2
        ```
    - 运行
        
        运行spark环境
        ```
        cd /usr/local/spark/
        sbin/start-all.sh
        # 登陆http://node-1:8080/查看
        ```  
        写python wordcount.py程序
        ```
        mkdir /usr/local/spark/mycode
        mkdir /usr/local/spark/mycode/python
        
        cd /usr/local/spark/mycode/python
        vim WordCount.py
        # 写入如下代码
        from pyspark import SparkConf, SparkContext
        conf = SparkConf().setMaster("local").setAppName("My App")
        sc = SparkContext(conf = conf)
        logFile = "file:///usr/local/spark/README.md"
        logData = sc.textFile(logFile, 2).cache()
        numAs = logData.filter(lambda line: 'a' in line).count()
        numBs = logData.filter(lambda line: 'b' in line).count()
        print('Lines with a: %s, Lines with b: %s' % (numAs, numBs))
        
        ```
        运行wordcount代码
        ```
        cd /usr/local/spark/
        bin/spark-submit  /usr/local/spark/mycode/python/WordCount.py
        ```
        在集群中运行pyspark
        ```
        # 向hdfs添加Readme文件
        hadoop fs -put /usr/local/spark/README.md /
        # 运行pyspark
        bin/pyspark  --master  spark://node-1:7077
        # 在pyspark运行以下测试代码
        textFile = sc.textFile("hdfs://node-1:9000/README.md")
        textFile.count()
        textFile.first()
        # 提交一个spark程序到集群，会产生的进程
        # SparkSubmit (Driver) 提交任务
        # Executor 执行真正计算任务

        ```
       

### 如果配置还是出错，去百度网盘下载链接：https://pan.baidu.com/s/1rRvBzwr5YUAnFyajutFrjg 提取码：u8r2 

### 大小15多G，可能要充会员下载，下载完导入镜像然后把上述有关ip的地方全部更改一下就可以了

        
