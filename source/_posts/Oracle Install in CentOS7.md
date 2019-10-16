---
title: 再次入魂——小白在CentOS7.6环境下部署Oracle11g服务器并使用Windows客户端连接
tags: CentOS,Linux
---
# 再次入魂——小白在CentOS7.6环境下部署Oracle11g服务器并使用Windows客户端连接





# 1.学习几个基本Linux终端指令  
　　由于Linux和Windows的软件安装方式有很大的不同，所以基本上都是需要在终端上敲命令行代码实现的。  
　　一、cd（change directory）切换目录  
　　`$cd　　　　　直接回到根目录`  
　　`$cd ..　　　　回到上级目录`  
　　`$cd 路径名　　切换至该路径`  
　　二、ls　查看文件或目录  
　　`$ls　　　　　查看当前目录的文件与目录`   
　　`$cd 路径名　　查看该路径下的文件与目录`  
　　ls命令常用于检查目录下的文件及目录。这样我们就不用费力地去找文件管理器下的文件而是可以直接在终端中查看。当我们不确定自己创建的目录是否成功时可以轻松地使用ls查看。  
　　三、su 和 sudo  
　　有些时候一些指令需要我们拥有管理员权限方能执行，这个时候这两个指令就非常重要。  
　　`su`  
　　进入root模式。su命令执行后，你需要在终端输入root密码，在终端中，密码是不显示的，输完直接敲回车即可。这是系统root用户，拥有很高的权限。值得注意的是，Ubuntu刚安装好是没有激活root的，需要我们手动激活。这时我们需要sudo指令。  
　　`sudo`  
　　在指令的前面加上sudo 即可在管理员模式下执行指令。当然，密码还是要输的，这里输的密码是你的系统用户密码（开机密码）而不是root密码。  
　　`sudo passwd root`  
　　执行该指令即可设定root密码。  
　　四、vim 文档编辑  
　　这个指令我们会在环境变量及配置文档的修改编辑中遇到。系统没有的话可在root模式中通过指令安装  
　　`yum -y install vim`  
　　安装好之后使用指令` vim xxx`进入xxx文件。键盘敲击'i'即可进入insert模式。编辑完毕后按'esc',在键盘上输入 **':wq'** 保存并退出。记住一定要加’冒号‘。不想保存的话用':q'。
# 2.安装Java并配置环境变量
　　由于Oracle是基于Java环境的数据库，所以必须要配置Java方可运行。CentOS如果你装了图形界面的话应该是自带Java的。但是没有安装jdk，如果你不需要安装sql developer(一款可视化Oracle操作工具)。那么这一步可以略过不看。   
　　一、卸载系统自带JDK  
　　依次执行如下指令  
　　`rpm -qa | grep java`  
　　命令说明：  
　　rpm 　　管理套件  
　　-qa 　　使用询问模式，查询所有套件  
　　grep　　查找文件里符合条件的字符串  
　　java 　　查找包含java字符串的文件
　　将除了.noarch文件的Java文件全删了。进入root用户，输入  
　
```
rpm -e --nodeps java-1.7.0-openjdk-1.7.0.111-2.6.7.8.el7.x86_64
rpm -e --nodeps java-1.8.0-openjdk-1.8.0.102-4.b14.el7.x86_64
rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.102-4.b14.el7.x86_64
rpm -e --nodeps java-1.7.0-openjdk-headless-1.7.0.111-2.6.7.8.el7.x86_64 
```
　　命令介绍：
　　-e	  				删除指定套件
　　--nodeps		不验证套件档的相互关联性
　　检查一下是否删除成功
　　`java -version`  
　　若未找到Java命令代表删除成功，若没有成功可用`yum -y remove`指令删一下。
　　二、安装JDK
　　从各种网站上找到你所需要的JDK版本并下载，这个看你自己，我下的是从官网上找的1.8。下载完毕打开安装目录，在目录中打开终端。将文件复制到/usr/java目录中并解压。
　　```
　　su
　　cp 安装包文件名 /usr/java
　　tar -zxvf 文件名
　　rm -f 安装包文件名
　　```
　　三、JDK环境变量的配置  
　　通过指令  
　　`sudo vim /etc/profile`  
　　打开profile文件，此文件存储Linux中的各种环境变量，敲击'i'在文件末尾加入以下内容：
```
export JAVA_HOME=/usr/java/jdk1.8.0_144

export CLASSPATH=.:${JAVA_HOME}/jre/lib/rt.jar:${JRE_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar

export PATH=$PATH:${JAVA_HOME}/bin: 
```
如何保存并退出在上一章节已经有提到，以后章节不再赘述。保存完毕后我们先要更新一下系统参数`source /etc/profile`在终端测试一下是否安装成功。依次输入指令。  
```　　
echo %JAVA_HOME   
java -version
```
若出现我们添加进profile文件中的路径以及正确的java版本号则安装成功。若未出现，可能你需要检查一下profile文件或者java安装包。
# 3.安装前的准备工作  
在进行各种依赖的下载还有乱七八糟的事之前，我们先把Oracle的安装包下载任务开始起来，这样就不需要多等一段时间了。Oracle11g的安装包在官网很容易找到，注意一定要将两个压缩包全部下载下来。最好还能定点下载到一个**指定的目录**里。本人就是将其下载至主文件夹下的**Oracle**文件夹下。这样方便后续的操作。  
注意：此处所用的指令都需要root老大哥才能完成。出错的时候先看看是不是权限的问题。

## 一、用户组创建并更改OS系统标志
运行下列指令通过命令行使用root模式创建用户和用户组
```
su
groupadd oinstall
groupadd dba
useradd -g oinstall -g dba -m oracle
```
查询用户组是否成功加入到用户组
```
groups oracle
```
设置oracle用户登录密码
`passwd oracle`
由于Oracle默认只支持redhat，不支持Ubuntu、centos等其他Linux系统，所以将系统标志修改成RedHat-7
`vim /etc/redhat-release`
内容修改为`redhat-7`。
关闭防火墙。
`systemctl stop firewalld`
其实可以仅仅开放你所需要的端口。笔者在这里使用的是9999端口，Oracle默认的是1521端口。
```
firewall-cmd --zone=public --add-port=9999/tcp --permanent  

firewall-cmd --reload
```
关闭selinux（重启后生效）
```
vim /etc/selinux/config
//修改SELINUX=disabled并去掉#
```
重启并进入oracle用户
`reboot`
注意：在生产环境下不可关闭防火墙和selinux。
## 二、创建Oracle数据库安装目录
创建目录
```
mkdir -p /data/oracle
mkdir -p /data/oraInventory
mkdir -p /data/database
```
注意：这里使用的都是绝对路径，即在前面有个‘/’。
接下来将这些文件夹的使用权限都赋给我们刚刚创建的oracle用户。
```
cd /data
ls
chown -R oracle:oinstall /data/oracle
chown -R oracle:oinstall /data/oraInventory
chown -R oracle:oinstall /data/database
```
## 三 、依赖包的安装  
　　Linux不同于Windows，这些东西都需要我们自己手动去安装。要装的东西多的要死，这里用yum指令进行批量安装，要稍等一会儿。  
`yum install binutils-2.* compat-libstdc++-33* elfutils-libelf-0.* elfutils-libelf-devel-* gcc-4.* gcc-c++-4.* glibc-2.* glibc-common-2.* glibc-devel-2.* glibc-headers-2.* ksh-2* libaio-0.* libaio-devel-0.* libgcc-4.* libstdc++-4.* libstdc++-devel-4.* make-3.* sysstat-7.* unixODBC-2.* unixODBC-devel-2.* pdksh* ` 
## 四、配置系统内核参数  
　　这里我们可以直接开两个终端来配，查一个配一个比较方便。  
　　新建一个终端键入指令`sudo vim /etc/sysctl.conf`。  
　　原终端按下列指令依次查出对应参数的值。  
```
/sbin/sysctl -a | grep sem

/sbin/sysctl -a | grep file-max

/sbin/sysctl -a | grep aio-max

/sbin/sysctl -a | grep ip_local_port_range

/sbin/sysctl -a | grep rmem_default

/sbin/sysctl -a | grep rmem_max

/sbin/sysctl -a | grep wmem_default

/sbin/sysctl -a | grep wmem_max

/sbin/sysctl -a | grep shmall

/sbin/sysctl -a | grep shmmax

/sbin/sysctl -a | grep shmmni　　
```
　　再将值填入对应的参数中并添加至新终端的'sysctl.conf'的末尾并保存退出。确保填入的值尽量是默认的内核参数，但是不能少于下列示例。　　
```
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.rp_filter = 1
fs.file-max = 6815744 #设置最大打开文件数
fs.aio-max-nr = 1048576
kernel.shmall = 2097152 #共享内存的总量，8G内存设置：2097152*4k/1024/1024
kernel.shmmax = 2147483648 #最大共享内存的段大小
kernel.shmmni = 4096 #整个系统共享内存端的最大数
kernel.sem = 250 32000 100 128
net.core.rmem_default = 262144
net.core.rmem_max= 4194304
net.core.wmem_default= 262144
net.core.wmem_max= 1048576
net.ipv4.ip_local_port_range = 1000 65500 #可使用的IPv4端口范围
```
更新内核参数`sysctl -p`。  
## 五、配置Oracle环境变量
　　现在我们要配置Oracle的环境变量。`sudo vim /home/oracle/.bash_profile`  。将下列内容添加至文件后面
```
export ORACLE_BASE=/data/oracle

export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1

export ORACLE_SID=orcl

export ORACLE_UNQNAME=orcl

export ORACLE_TERM=xterm

export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib #添加系统环境变量

export LANG=C #防止安装过程出现乱码

export NLS_LANG=AMERICAN_AMERICA.ZHS16GBK  #设置Oracle客户端字符集，必须与Oracle安装时设置的字符集保持一致

export PATH=${PATH}:${ORACLE_HOME}/bin/:/usr/sbin
```
保存并退出后更新一下环境变量`source /home/oracle/.bash_profile`。
## 三、正式安装
　　将安装包解压至指定目录。  
```
unzip linux.x64_11gR2_database_1of2.zip -d /data/database

unzip linux.x64_11gR2_database_2of2.zip -d /data/database

chown -R oracle:oinstall /data

cd /data/database/database

ls

./runInstaller
```
这样我们就进入了安装Oracle的图形界面了。
注意：这样做有的情况会出现提示条只出现白条的问题，这样我们就无法进行下一步了，甚至连退出都够呛。笔者就出现过这种情况。这个我们参考https://blog.csdn.net/ykyorky/article/details/79259424?utm_source=blogxgwz6中的解决方案。使用指令
`./runInstall -jreLoc JDK安装路径`
进行安装
->注：以下除了密码之外未提及支之处全部选择默认值即可。密码这个自行决定。
->在Configure Security Updates中去掉“I wish”，邮箱也不用填直接下一步。弹出警告框也不用管。下面同理不多赘述。  
->在Installation option中选择install database software only
->在Grid Installation中选择single instance database installation  
->在Product Languages上可以添加简体中文（simplified-chinese）  
->在Select Database Edition选择企业版（enterprise）
->在Typical Installation中路径选择刚才安装包解压的目录，笔者的目标目录在上面已经写好，不多赘述。输入密码并确认。点击下一步，报错不管它。  
->第十二步，选择上面的character sets进入界面 选择Use Unicode(AL32UTF8),然后选择上面Sample Schemas,进入界面后将选项打钩  
->在Prerequis like Checks中勾选 ignore All。因为我们之前已经安装好所有的依赖了，所以只要忽略即可。这里最好还要新建一个终端跑一下。
```
cd /tmp/CVU_11.2.0.1.0_用户名

sudo ./runfixup.sh
```
->点击Finish开始安装。
->安装时会报错,新建终端敲一下指令。
```
su

yum -y install glibc-static

vim /data/oracle/product/11.2.0/db_1/ctx/lib/ins_ctx.mk
```
将
`ctxhx: $(CTXHXOBJ)
       $(LINK_CTXHX) $(CTXHXOBJ) $(INSO_LINK)`
修改为：
`ctxhx: $(CTXHXOBJ)
       -static $(LINK_CTXHX) $(CTXHXOBJ) $(INSO_LINK) /usr/lib64/libc.a`
接着再改一下ins_emagent.mk文件
```
cd /data/oracle/product/11.2.0/dbhome_1/sysman/lib

yum -y install gedit

gedit ins_emagent.mk
```
在打开的文件中找到`$(SYSMANBIN)emdctl`，将冒号后的值改为`$(MK_EMAGENT_NMECTL -lnnz11)`。在gedit中可以使用Ctrl+f进行查找。修改完毕后保存退出。
点击retry。  
如果出现很多权限问题可自行百度，笔者碰到过一个坑但是解决方法忘了，大致就是改一下配置文件或者授予文件夹权限之类的。
->后面一直ok就行。

## 四、配置监听并创建数据库实例测试
首先检查一下参数
```
echo $ORACLE_BASE

echo $ORACLE_HOME

echo $PATH
```
如果不对的话你需要看一下之前的环境变量设置。仔细查看你的路径名，包括用户名是不是自己的。参数对的话就可以启动监听了。  
## 1）配置监听
我们使用命令`netca`进行监听的设定。
注意：如果你所使用的端口号在步骤3.1中没有打开，那么是无法创建监听的。这里面基本没什么不懂的，看得懂英文就行。只不过要注意在选择协议的时候选择TCP。出错的话可能是环境变量的问题或者是文件夹权限的问题，问题都不是很大。

## 2）创建一个Oracle数据库实例
如果你看了别的教程在安装阶段就已经创建了数据库实例，那么可以跳过本步骤。这里说的是没有创建数据库实例的情况。执行`dbca`命令进入图形界面。点击next即可进入下一步。![在这里插入图片描述](https://img-blog.csdnimg.cn/20190408233533957.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNjcwMzA0,size_16,color_FFFFFF,t_70)
选择“create a database”
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190408233707292.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNjcwMzA0,size_16,color_FFFFFF,t_70)
选择“custom dataase”
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190408234023515.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNjcwMzA0,size_16,color_FFFFFF,t_70)
填入之前在环境变量文件中写的sid，如果上面你是直接复制的话那么填入`orcl`即可，想换的话也行，不过要把环境变量中的一起换掉。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190408234033157.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNjcwMzA0,size_16,color_FFFFFF,t_70)
后面笔者懒得叙述了，反正基本都能成功，觉得讲得不够详细可以看我最后一章参考的资料。出错按照错误代码百度相应的解决方案。笔者遇到的错误比较奇葩。提示的是我共享内存空间不足了，于是我找了很多解决方案，最简单的还是直接reboot重启（重启大法好啊）。
## 3）测试
新开一个终端，写入下列命令
```
sqlplus /nolog

conn / as sysdba
```
这样你就可以用sys超级管理员用户连接我们刚刚创建的数据库实例orcl了。
可以在SQL里做个小测试
```sql
select 1 from dual;
```
成功了那你的Oracle服务器就部署成功了。
# 4.使用Windows客户机连接服务器上的Oracle
## 一、创建用户
直至上面测试做完为止，我们的数据库用户还是只有sys这一个超级用户。我们做项目不可能只有一个用户的，这个时候我们要创建给客户用的用户。在SQL中写以下指令。(注意：SQL指令是要加分号的)。还要注意把括号内说明段删掉。
```sql
create user njust2048(用户名)identified by admin(密码);

grant resource,connect,dba to cs2048;

select * from user_sys_privs;
```
对于普通用户：授予connect, resource权限。   
对于DBA管理用户：授予connect，resource, dba权限。  
DBA: 拥有全部特权，是系统最高权限，只有DBA才可以创建数据库结构。sysdba角色之外权限最大的角色。   
RESOURCE:拥有Resource权限的用户只可以创建实体，不可以创建数据库结构。   
CONNECT:拥有Connect权限的用户只可以登录Oracle，不可以创建实体，不可以创建数据库结构。
## 二、使用创建的用户连接数据库
这里有两种方案可以选择。
### 1）使用Oracle SQL developor
这种图形界面可以分为一个大类。有兴趣或者要求的读者可以寻找其他的软件。使用Oracle SQL developer的话必须要安装jdk才行。从Oracle官网上下载安装包（对应你客户机的jdk版本）并安装。进入界面。  点击加号新建连接。连接名自定义，用户名和密码使用我们刚刚创建好的用户和密码。主机名填写服务器域名。sid可以在服务器使用`select instance_name from v$instance;`查询。端口号为默认的1521。点击测试可以测试连接可靠性。测试成功后连接即可。
### 2）使用控制台连接数据库
在Windows下打开cmd或者power shell。执行以下命令。
`sqlplus cs2048/2048@192.168.1.113:1521/orcl`  
这样你就可以连接至服务器的数据库。注意，这里的参数顺序分别是：  
cs2048（用户名）；2048（密码）；192.168.1.113（服务器IP）；orcl（服务名）
# 参考资料：
https://www.cnblogs.com/muhehe/p/7816808.html
https://blog.csdn.net/ykyorky/article/details/79259424?utm_source=blogxgwz6
https://www.cnblogs.com/buxingzhelyd/p/7865194.html