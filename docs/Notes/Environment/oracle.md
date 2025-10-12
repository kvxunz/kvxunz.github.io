## 更新系统

首先，确保你的系统是最新的。


```sh
sudo dnf update -y
sudo dnf install -y oracle-database-preinstall-19c
```
## 安装gcc/g++ 11
1. 获取root权限
  - 确保你有正确的权限，可以通过 sudo 运行命令

2. 启用开发工具的yum仓库
  - 为了安装开发相关的软件包以及特定版本的 GCC，需要启用开发工具组：

  ```sh
  sudo dnf groupinstall "Development Tools"
  ```

3. 安装 GCC/G++ 11
  - 使用 dnf 安装特定版本的 GCC 和 G++：

  ```sh
  sudo dnf install gcc-toolset-11-gcc gcc-toolset-11-gcc-c++
  ```

4. 启用 GCC/G++ 11
  - 安装完成后，启用 GCC 11 的工具链：

  ```sh
  scl enable gcc-toolset-11 bash
  ```

5. 永久生效

```sh
nano ~/.bashrc
```
```sh
source /opt/rh/gcc-toolset-11/enable
source ~/.bashrc
```
6. 验证安装

```sh
gcc --version
g++ --version
which gcc
```

7. 使oracle用户可以访问gcc/g++ 11

```sh
[root@kv4 ~]# which gcc
```
```sh
/opt/rh/gcc-toolset-11/root/usr/bin/gcc
```
```sh
su - oracle
```
```sh
echo 'export PATH=/opt/rh/gcc-toolset-11/root/usr/bin:$PATH' >> ~/.bashrc
```
```sh
source ~/.bashrc
```
```sh
gcc --version
```


## 创建必要的目录

创建一个目录来存放 Oracle Database 软件和数据文件。
```sh
sudo mkdir -p /u01/app/oracle/product/19.0.0/dbhome_1
sudo chown -R oracle:oinstall /u01
sudo chmod -R 775 /u01
```

## 设置环境变量

编辑 oracle 用户的环境变量文件（.bash_profile 或 .bashrc）

```sh
sudo su - oracle
nano ~/.bashrc
```

添加以下行：
```sh
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/19.0.0/dbhome_1
export ORACLE_SID=ORCLCDB
export PATH=$ORACLE_HOME/bin:$PATH
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
export CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib

```

保存并退出编辑器，然后加载新的配置：
```sh
source ~/.bashrc
```

## 解压安装文件

将下载的 LINUX.ARM64_1919000_db_home.zip 文件上传到服务器（例如使用 scp 或其他文件传输工具），然后解压：

```sh
wget -O LINUX.ARM64_1919000_db_home.zip ''
unzip LINUX.ARM64_1919000_db_home.zip -d $ORACLE_HOME

```
## 扩展交换区
为了扩展交换区到 16 GiB，可以进行以下步骤：
关闭并移除当前的交换区文件。
创建一个新的交换区文件。
设置新的交换区文件。
启用新的交换区。
永久启用新的交换区。
以下是具体步骤：
1. 关闭并移除当前的交换区文件：
```sh
sudo swapoff /.swapfile
sudo rm /.swapfile
```
2. 创建一个新的交换区文件：
这里创建一个 16 GiB 的交换区文件
```sh
sudo dd if=/dev/zero of=/.swapfile bs=1G count=16
```
3. 设置新的交换区文件：
```sh
sudo chmod 600 /.swapfile
sudo mkswap /.swapfile
```
4. 启用新的交换区：
```sh
sudo swapon /.swapfile
```
5. 确认新的交换区：

使用 swapon -s 或 free -h 查看新的交换区配置，确保新的交换区已启用：
```sh
swapon -s
free -h
```
6. 永久启用新的交换区：
为了使新的交换区在系统重启后依然有效，需要修改 /etc/fstab 文件：
```sh
echo '/.swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```
这样做之后，系统应该会有一个 16 GiB 的交换区满足 Oracle 数据库的需求。


## 运行安装程序
注意：由于没有图形界面，可能需要使用 silent mode 来进行安装。你需要一个响应文件来提供安装选项。可以使用默认的响应文件模板并进行编辑：


```sh
cp $ORACLE_HOME/install/response/db_install.rsp /tmp/db_install.rsp
nano /tmp/db_install.rsp
```

根据你的需求编辑响应文件（如设置 ORACLE_BASE、ORACLE_HOME 等），确保以下参数正确配置：


```sh
oracle.install.option=INSTALL_DB_SWONLY
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/u01/app/oraInventory
ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
ORACLE_BASE=/u01/app/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.OSDBA_GROUP=dba
oracle.install.db.OSBACKUPDBA_GROUP=backupdba
oracle.install.db.OSDGDBA_GROUP=dgdba
oracle.install.db.OSKMDBA_GROUP=kmdba
oracle.install.db.OSRACDBA_GROUP=racdba

```

然后以静默模式运行安装程序：

```sh
cd $ORACLE_HOME
./runInstaller -silent -responseFile /tmp/db_install.rsp

```

## 执行 root 脚本

安装完成后，以 root 用户身份执行两个脚本：

```sh
sudo /u01/app/oraInventory/orainstRoot.sh
sudo /u01/app/oracle/product/19.0.0/dbhome_1/root.sh

```




## 启动监听


1. 确认网络接口和IP地址绑定
确保服务器的网络接口配置包含 132.226.31.158公共IP地址。
查看当前网络接口和IP地址:
```sh
   ip addr show
```
如果没有显示 132.226.31.158，你需要手动将其添加到网络接口。
2. 手动绑定IP地址:
以下是临时绑定IP地址的命令（重启后失效):
```sh
   sudo ip addr add /24 dev enp0s6
   sudo ip link set dev enp0s6 up
```
注: 确保 enp0s6 是正确的网络接口名称。
3. 永久配置网络接口：
如果需要永久绑定IP地址，可以编辑网络配置文件。这通常因不同的操作系统版本而异。以 CentOS 为例：
```sh
   sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s6
```
添加以下条目：
```sh
   IPADDR=132.226.31.158
   NETMASK=255.255.255.0
```
保存并重启网络服务:
```sh
sudo systemctl restart network
```



```sh
ifconfig
nano $ORACLE_HOME/network/admin/listener.ora


SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = ORCLCDB)
      (SID_NAME = ORCLCDB)
      (ORACLE_HOME = /u01/app/oracle/product/19.0.0/dbhome_1)
    )
  )

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 129.153.209.187)(PORT = 1521))
      (ADDRESS = (PROTOCOL = TCP)(HOST = 10.0.0.18)(PORT = 1521))
    )
  )

telnet 129.153.209.187 1521
[root@kh ~]# nc -zv 132.226.31.158 1521
[root@kh ~]#    sudo firewall-cmd --list-all
[root@kh ~]#    sudo firewall-cmd --add-port=1521/tcp --permanent
[root@kh ~]# sudo firewall-cmd --reload

```
```sh
netca（Oracle Network Configuration Assistant） 是一个图形用户界面工具，需要设置 DISPLAY 环境变量以便在 GUI 环境中运行。但由于目前是在命令行环境下，你可以修改监听器配置文件来手动配置监听器，或者使用静默模式运行 netca 工具。
以下是两种解决办法：
方法 1: 静默模式运行 netca
你可以在命令行中使用静默模式运行 netca 工具来配置监听器。
执行以下命令来静默运行 netca:

netca -silent -responseFile $ORACLE_HOME/assistants/netca/netca.rsp
方法 2: 手动配置监听器
如果你希望手动配置监听器，可以修改 listener.ora 文件。通常，该文件位于 $ORACLE_HOME/network/admin/ 目录中。
创建或编辑文件 $ORACLE_HOME/network/admin/listener.ora，内容如下：

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = your_hostname)(PORT = 1521))
    )
  )
记得将 your_hostname 替换为你的实际主机名或IP地址。
然后，启动监听器：

lsnrctl start
验证监听器配置
你可以使用以下命令来检查监听器的状态，以确保其正确配置并运行：

lsnrctl status

```


## 创建数据库

创建一个响应文件 /tmp/dbca.rsp 并进行编辑：

```sh
gdbName=ORCLCDB
sid=ORCLCDB
templateName=General_Purpose.dbc
characterSet=AL32UTF8
memoryPercentage=40
emConfiguration=DBEXPRESS
emExpressPort=5500
systemPassword=Alertzk-1
sysPassword=Alertzk-1
```
使用 dbca 创建数据库，同样可以使用静默模式：
```sh
dbca -silent -createDatabase -responseFile /tmp/dbca.rsp
```

## 验证安装

确认数据库已经正确创建和启动：
```sh
sqlplus / as sysdba
```
在 SQL*Plus 中运行：
```sh
SELECT name, open_mode FROM v$database;
```













## 删除

1. 停止数据库实例
如果数据库正在运行，首先需要停止数据库实例：
```sh
sqlplus / as sysdba
```
在 SQL*Plus 中执行以下命令：
```sh
shutdown immediate;
exit;
```
2. 删除数据库
使用 Oracle DBCA 工具可以简单地删除数据库：
```sh

dbca -silent -deleteDatabase -sourceDB ORCLCDB
```
3. 删除相关文件和目录
可以手动删除与数据库实例相关的物理文件和目录：
```sh

rm -rf /u01/app/oracle/oradata/ORCLCDB
rm -rf /u01/app/oracle/flash_recovery_area/ORCLCDB
rm -rf /u01/app/oracle/admin/ORCLCDB
```
确保根据你的具体 Oracle 环境路径修改这些命令。
4. 从 /etc/oratab 文件中删除对应的条目
编辑 /etc/oratab 文件，删除与 ORCLCDB 相关的行：
```sh

nano /etc/oratab
```
找到并删除相应的条目：
```sh
ORCLCDB:/u01/app/oracle/product/19.0.0/dbhome_1:N
```


## 数据库实例注册

1. 检查和启动数据库实例
确保数据库实例是启动状态，并且正确地注册到了监听器中。
```sh

sqlplus / as sysdba

SQL> startup
```
2. 设置本地监听器变量
确保 LOCAL_LISTENER 参数在数据库实例中正确配置。
```sh

sqlplus / as sysdba

SQL> alter system set LOCAL_LISTENER='(ADDRESS=(PROTOCOL=TCP)(HOST=129.146.30.114)(PORT=1521))';
SQL> alter system register;
```
这将手动注册数据库实例到监听器中，并确保监听器知道应该监视哪些服务。
3. 从监听器日志文件中确认注册情况
查看监听器日志文件，确认服务是否注册成功:
```sh

tail -f /u01/app/oracle/diag/tnslsnr/kv4/listener/alert/log.xml
```
你应能看到服务注册的信息。
再次检查监听器状态
重新检查监听器状态，确认服务是否注册成功：
```sh
lsnrctl status
```



## 编辑 SSH 配置文件
使用你喜欢的文本编辑器编辑 /etc/ssh/sshd_config 文件。例如：
```sh
sudo nano /etc/ssh/sshd_config
```
完成配置文件修改后，重启 SSH 服务以使更改生效。
```sh
sudo systemctl restart sshd
```
