---
title: 一发入魂——小白在Ubuntu16环境下部署Oracle服务器并使用Windows客户端连接
tags: linux
---
# **一发入魂——小白在Ubuntu16环境下部署Oracle服务器并使用Windows客户端连接**
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
　　这个指令我们会在环境变量及配置文档的修改编辑中遇到。系统没有的话可以通过指令安装  
　　`sudo apt-get install vim`  
　　安装好之后使用指令`sudo vim xxx`进入xxx文件。键盘敲击'i'即可进入insert模式。编辑完毕后按'esc',在键盘上输入 **':wq'** 保存并退出。记住一定要加’冒号‘。不想保存的话用':q'。
# 2.安装JDK并配置环境变量
　　由于Oracle是基于Java环境的数据库，所以必须要配置JDK方可运行。我们首先打开终端。   
　　一、JDK安装  
　　依次执行如下指令  
　　`sudo apt-get update`  
　　`sudo apt-get install openjdk-8-jdk`   
　　这里安装的是JDK8，大家可以根据实际情况选用所需的JDK版本。  
　　二、JDK环境变量的配置  
　　通过指令  
　　`sudo vim /etc/profile`  
　　打开profile文件，此文件存储Linux中的各种环境变量，后面的Oracle环境变量也将在此编辑。敲击'i'在文件末尾加入以下内容：
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64

export JRE_HOME=${JAVA_HOME}/jre

export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib

export PATH=${JAVA_HOME}/bin:$PATH ```
如何保存并退出在上一章节已经有提到，以后章节不再赘述。保存完毕后我们先要更新一下系统参数`source /etc/profile`在终端测试一下是否安装成功。依次输入指令。  
```　　
echo %JAVA_HOME   
java -version
```
若出现我们添加进profile文件中的路径以及正确的java版本号则安装成功。若未出现，可能你需要检查一下profile文件或者java安装包。
# 3.Oracle安装  
在进行各种依赖的下载还有乱七八糟的事之前，我们先把Oracle的安装包下载任务开始起来，这样就不需要多等一段时间了。Oracle11g的安装包在官网很容易找到，注意一定要将两个压缩包全部下载下来。最好还能定点下载到一个**指定的目录**里。本人就是将其下载至主文件夹下的**Oracle**文件夹下。这样方便后续的操作。  
## 一、依赖包的安装  
　　Linux不同于Windows，这些东西都需要我们自己手动去安装。要装的东西多的要死，笔者参考了一下网上的代码，来自Linux社区的rogear将其整合了一下，通过一句指令安装所有的依赖包。 因为包太多了，所以安装起来要稍微等一会儿。  
`sudo apt-get -y install automake autotools-dev binutils bzip2 elfutils expat gawk gcc gcc-multilib g++-multilib lib32ncurses5 lib32z1 ksh less lib32z1 libaio1 libaio-dev libc6-dev libc6-dev-i386 libc6-i386 libelf-dev libltdl-dev libxm4 libodbcinstq4-1 libodbcinstq4-1:i386 libpth-dev libpthread-stubs0-dev libstdc++5 make openssh-server rlwrap rpm sysstat unixodbc unixodbc-dev unzip x11-utils zlibc`  
　　笔者亲测没有问题，要问为什么，因为我也是小白！有两个依赖是32位的，需要换源安装。也不是太麻烦，依次执行以下指令即可。
```
su

cd /etc/apt/sources.list.d

echo "deb http://old-releases.ubuntu.com/ubuntu/ raring main restricted universe multiverse" > ia32-libs-raring.list

apt update

apt-get -y install lesstif2 lesstif2-dev

rm -rf ia32-libs-raring.list

apt update

exit
```
　　如果出现什么问题，那就将前两步调换一下位置。Linux中的依赖包有的是相互依赖的，笔者小白一枚也没搞清楚。  
## 二、配置环境  
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
　　再将值填入对应的参数中并添加至新终端的'sysctl.conf'的末尾并保存退出。　　
```
fs.aio-max-nr = 65536

fs.file-max = 242293

kernel.shmall = 18446744073692774399

kernel.shmmax = 18446744073692774399

kernel.shmmni = 4096

kernel.sem = 32000 1024000000 500 32000

net.ipv4.ip_local_port_range = 32768 60999

net.core.rmem_default = 212992

net.core.rmem_max = 212992

net.core.wmem_default = 212992

net.core.wmem_max = 212992
```
更新内核参数`sysctl -p`。  
添加用户的内核限制
```
cd 
cd /etc/security
sudo vim limits.conf
```
添加以下内容至文件末尾（'end of file'后）。用户名就是你登录Ubuntu的用户名，比如我的是njust。
```
用户名 soft nproc 2047

用户名 hard nproc 16384

用户名 soft nofile 1024

用户名 hard nofile 65536

用户名 soft stack 10240
```
　　现在我们要配置Oracle的环境变量。`sudo vim /etc/profile`  。将下列内容添加至文件后面
```
export ORACLE_BASE=你的Oracle路径,笔者是/home/njust/Oracle

export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1

export ORACLE_SID=orcl

export ORACLE_UNQNAME=orcl

export NLS_LANG=.AL32UTF8

export PATH=${PATH}:${ORACLE_HOME}/bin/:$ORACLE_HOME/lib64
```
保存并退出后更新一下环境变量`source /etc/profile`。
　　最好检查一下/etc/pam.d/login，增加以下行（有了就不用增加了）： session required pam_limits.so。    
　　检查/etc/pam.d/su，没有以下行就自己加上： session required pam_limits.so。  
　　由于Oracle默认不支持Ubuntu，所以我们需要通过以下命令欺骗一下Oracle安装程序。
```
sudo mkdir /usr/lib64

sudo ln -s /etc /etc/rc.d

sudo ln -s /lib/x86_64-linux-gnu/libgcc_s.so.1 /lib64/

sudo ln -s /usr/bin/awk /bin/awk

sudo ln -s /usr/bin/basename /bin/basename

sudo ln -s /usr/bin/rpm /bin/rpm

sudo ln -s /usr/lib/x86_64-linux-gnu/libc_nonshared.a /usr/lib64/

sudo ln -s /usr/lib/x86_64-linux-gnu/libpthread_nonshared.a /usr/lib64/

sudo ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /lib64/

sudo ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /usr/lib64/

su

echo 'Red Hat Linux release 5' > /etc/RedHat-release

exit
```
## 三、正式安装
　　将安装包解压至指定目录，本人是直接在/Home/Oracle下解压的。解压后会生成一个database目录。  
```
cd /Oracle

unzip linux.x64_11gR2_database_1of2.zip

unzip linux.x64_11gR2_database_2of2.zip

cd database/

./runInstaller
```
这样我们就进入了安装Oracle的图形界面了。如果发现安装界面乱码可以执行命令  
```
export NLS_LANG=AMERICAN_AMERICA.UTF8

export LC_ALL=C  
```
->注：以下除了密码之外未提及支之处全部选择默认值即可。密码这个自行决定。
->在Configure Security Updates中去掉“I wish”，邮箱也不用填直接下一步。弹出警告框也不用管。下面同理不多赘述。  
->在System Class中选择Server Class。  
->选择Advanced install  
->在Product Languages上可以添加简体中文（simplified-chinese）  
->在Typical Installation中路径选择刚才安装包解压的目录，笔者的目录就是/Home/Oracle。输入管理员密码并确认。点击下一步，报错不管它。  
->第十二步，选择上面的character sets进入界面 选择Use Unicode(AL32UTF8),然后选择上面Sample Schemas,进入界面后将选项打钩  
->在Prerequis like Checks中勾选 ignore All。因为我们之前已经安装好所有的依赖了，所以只要忽略即可。这里最好还要新建一个终端跑一下。
```
cd /tmp/CVU_11.2.0.1.0_用户名

sudo ./runfixup.sh
```
->点击Finish开始安装。
->安装至68%时会报错,第一个错误（有关ins_ctx.mk）直接continue。从第二个错开始我们就要干活儿了。  
->第二个错误，我们需要新建终端执行以下指令
```
cd /Oracle/product/11.2.0/dbhome_1/sysman/lib

sudo apt install gedit

gedit ins_emagent.mk
```
在打开的文件中找到`$(SYSMANBIN)emdctl`，将冒号后的值改为`$(MK_EMAGENT_NMECTL -lnnz11)`。在gedit中可以使用Ctrl+f进行查找。修改完毕后保存退出。点击retry。  
->我们发现又会报错，没关系，依次执行以下指令即可。执行完retry即可。
```
sudo sed -i 's/^\(TNSLSNR_LINKLINE.*\$(TNSLSNR_OFILES)\) \(\$(LINKTTLIBS)\)/\1 -Wl,--no-as-needed \2/g' /home/rogear/tools/oracle11g/product/11.2.0/dbhome_1/network/lib/env_network.mk

sudo sed -i 's/^\(ORACLE_LINKLINE.*\$(ORACLE_LINKER)\) \(\$(PL_FLAGS)\)/\1 -Wl,--no-as-needed \2/g' /home/rogear/tools/oracle11g/product/11.2.0/dbhome_1/rdbms/lib/env_rdbms.mk

sudo sed -i 's/^\(\$LD \$LD_RUNTIME\) \(\$LD_OPT\)/\1 -Wl,--no-as-needed \2/g' /home/rogear/tools/oracle11g/product/11.2.0/dbhome_1/bin/genorasdksh

sudo sed -i 's/^\(\s*\)\(\$(OCRLIBS_DEFAULT)\)/\1 -Wl,--no-as-needed \2/g' /home/rogear/tools/oracle11g/product/11.2.0/dbhome_1/srvm/lib/ins_srvm.mk
```
->后面一直ok就行。

## 四、测试
首先检查一下参数
```
echo $ORACLE_BASE

echo $ORACLE_HOME

echo $PATH
```
如果不对的话你需要看一下之前的环境变量设置。仔细查看你的路径名，包括用户名是不是自己的。参数对的话就可以启动监听了。  
`lsnrctl start`  
如果不行的话，基本上还是环境变量的问题。启动服务
```
sqlplus /nolog

conn / as sysdba

startup
```
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
这种图形界面可以分为一个大类。有兴趣或者要求的读者可以寻找其他的软件。  
从Oracle官网上下载安装包（对应你客户机的jdk版本）并安装。进入界面。  点击加号新建连接。连接名自定义，用户名和密码使用我们刚刚创建好的用户和密码。主机名填写服务器域名。sid可以在服务器使用`select instance_name from v$instance;`查询。端口号为默认的1521。点击测试可以测试连接可靠性。测试成功后连接即可。
### 2）使用控制台连接数据库
在Windows下打开cmd或者power shell。执行以下命令。
`sqlplus sys/password@192.168.110.30/orcl as sysdba`  
# 参考资料：
https://blog.csdn.net/weixin_40461281/article/details/79784429   
https://www.linuxidc.com/Linux/2017-12/149797.htm
