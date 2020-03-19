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
        
        # 添加如下
        192.168.199.128    node-1
        192.168.199.129    node-2
        192.168.199.130    node-3
        ```
    - ssh 免密登陆，下载ssh  apt-get install openssh-server
        ```
        #对本机免密码登录：
        ssh-keygen -t rsa
        cd /root/.ssh
        cp id_rsa.pub authorized_keys
        
        # 三台机器之间的免密码登录：
        ssh-copy-id -i  目标主机名\
        例：ssh-copy-id -i node-2
        ```
        ```
        # 修改ssh，使本地也可ssh到node-1
        vim /etc/ssh/sshd_config 
        # 找到PermitRootLogin prohibit-password将该行改为PermitRootLogin yes，注意去掉前面的# 
        # 重启ssh
        service ssh restart
        ```
3. #### jdk安装
    - 点击[点击此链接](https://download.oracle.com/otn-pub/java/jdk/13.0.2+8/d4173c853231432d94f001e99d882ca7/jdk-13.0.2_linux-x64_bin.tar.gz?AuthParam=1583251205_0b11e0c4802087ccc2f0305dd154dba1)下载jdk到node-1里。
        ```
        # 创建目录
        sudo mkdir /usr/lib/jvm
        # 解压
        tar zxvf jdk-13.0.2_linux-x64_bin.tar.gz -C /usr/java/
        # 修改环境变量
        sudo vim ~/.bashrc
        # 末尾添加
        export JAVA_HOME=/usr/java/jdk-13.0.2  ## 这里要注意目录要换成自己解压的jdk 目录
        export JRE_HOME=${JAVA_HOME}/jre  
        export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib export PATH=${JAVA_HOME}/bin:$PATH  
        
        #  使环境变量马上生效

        source ~/.bashrc
        # 设置系统默认jdk 版本
        sudo update-alternatives --install /usr/bin/java java /usr/java/jdk-13.0.2/bin/java 300  
        sudo update-alternatives --install /usr/bin/javac javac /usr/java/jdk-13.0.2/bin/javac 300  
        sudo update-alternatives --install /usr/bin/jar jar /usr/java/jdk-13.0.2/bin/jar 300   
        sudo update-alternatives --install /usr/bin/javap javap /usr/java/jdk-13.0.2/bin/javap 300
        
        # 然后执行:
        sudo update-alternatives --config java
        
        # 测试
        java --version
        ```
