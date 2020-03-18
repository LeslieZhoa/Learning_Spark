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