# Before install CDH&Clouder amanager

<table class="tg">
  <tr>
    <th class="tg-xldj">physical slot</th>
    <th class="tg-0pky">hostname</th>
    <th class="tg-0pky">ip</th>
    <th class="tg-0pky">roles</th>
    <th class="tg-0pky">CentOS7.5_installed</th>
    <th class="tg-0pky">disk</th>
  </tr>
  <tr>
    <td class="tg-0pky">slot1</td>
    <td class="tg-0pky">n1</td>
    <td class="tg-0pky">10.66.27.61</td>
    <td class="tg-0pky">CM-server,CM-agent</td>
    <td class="tg-0pky">yes</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-0pky">slot2</td>
    <td class="tg-0pky">n2</td>
    <td class="tg-0pky">10.66.27.62</td>
    <td class="tg-0pky">CM-agent</td>
    <td class="tg-0pky">yes</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-0pky">slot3</td>
    <td class="tg-0pky">n3</td>
    <td class="tg-0pky">10.66.27.63</td>
    <td class="tg-0pky">CM-agent</td>
    <td class="tg-0pky">yes</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-0pky">slot4</td>
    <td class="tg-0pky">n4</td>
    <td class="tg-0pky">10.66.27.64</td>
    <td class="tg-0pky">CM-agent</td>
    <td class="tg-0pky">yes</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-0pky">slot7/8</td>
    <td class="tg-0pky">n5</td>
    <td class="tg-0pky">10.66.27.68</td>
    <td class="tg-0pky">CM-agent</td>
    <td class="tg-0pky">yes</td>
    <td class="tg-0pky">2.5T via iscsi_server 10.66.27.41(server not accessible)</td>
  </tr>
  <tr>
    <td class="tg-0pky">slot11</td>
    <td class="tg-0pky">n6</td>
    <td class="tg-0pky">10.66.27.71</td>
    <td class="tg-0pky">CM-agent</td>
    <td class="tg-0pky">yes</td>
    <td class="tg-0pky">2.5T via iscsi_server 10.66.27.41(server not accessible)</td>
  </tr>
  <tr>
    <td class="tg-0pky">slot12</td>
    <td class="tg-0pky">n7</td>
    <td class="tg-0pky">10.66.27.72</td>
    <td class="tg-0pky">CM-agent</td>
    <td class="tg-0pky">yes</td>
    <td class="tg-0pky">2.5T via iscsi_server 10.66.27.43</td>
  </tr>
  <tr>
    <td class="tg-0pky">slot14</td>
    <td class="tg-0pky">n8</td>
    <td class="tg-0pky">10.66.27.74</td>
    <td class="tg-0pky">CM-agent</td>
    <td class="tg-0pky">yes</td>
    <td class="tg-0pky">2.5T via iscsi_server 10.66.27.43</td>
  </tr>
</table>
>network_device: enp4s0f2
>
>Gateway:10.66.27.1; DNS:10.66.27.18

## 1. set hostname
Use command: hostnamectl set-hostname your_hostname 
```shell
hostnamectl set-hostname n[1-8]
```
## 2. set ssh public keys
### method 1:
```shell
#node n1:
vi /root/Documents/hostlist
10.66.27.61 n1
10.66.27.62 n2
10.66.27.63 n3
10.66.27.64 n4
10.66.27.68 n5
10.66.27.71 n6
10.66.27.72 n7
10.66.27.74 n8
:wq
cat hostlist >> /etc/hosts
ssh-keygen -t rsa

#for node n[2-8], generate public/private keys and copy to node n1(10.66.27.61)
ssh-keygen -t rsa
ssh-copy-id -i 10.66.27.61 #password needed

#Check authorized_keys in node n1, and send to other nodes
cat authorized_keys
for a in {2..8}; do scp /root/.ssh/authorized_keys root@n$a:/root/.ssh/authorized_keys ; done
#Modify /etc/hosts for node [2-8]
for a in {2..8}; do scp /root/Documents/hostlist root@n$a:/root/Documents/ ; done
#for a in {2..3}; do:
cat /root/Documents/hostlist >> /etc/hosts
```
### bash method to set hosts(**_not tested yet!_**):
set hostlist:
```shell
vi /root/Documents/hostlist
10.66.27.61 n1
10.66.27.62 n2
10.66.27.63 n3
10.66.27.64 n4
10.66.27.68 n5
10.66.27.71 n6
10.66.27.72 n7
10.66.27.74 n8
```
expect_host.sh: 
ssh login n[2-8], modify /etc/hosts with root privilege
```shell
#!/usr/bin/expect
# expect expect_hosts.sh 10.135.82.197 ubuntu ubuntu123456 

set timeout 1
#set host "10.66.27.61"
#set username "root"
#set password "qsmwwy123"
set host [lindex $argv 0]
set username [lindex $argv 1]
set password [lindex $argv 2]

spawn ssh $username@$host
expect {
    "Are you sure you want to continue connecting (yes/no)?" {send "yes\r"; exp_continue}
    "*password*" {send "$password\r"}
}
expect "*$"
send "sudo passwd root\r"
expect "*password*" {send "qsmwwy123\r"}
expect "*password*" {send "qsmwwy123\r"}

expect "*$"
send "su - root\r"
expect "Password:" {send "qsmwwy123\r"}

expect "*#"
send "cat /root/Documents/hostlist > /etc/hosts\r"
expect 
interact
expect EOF
```

root_hosts.sh: 
run this script on node n1
```shell
#!/bin/bash

nodes=($(awk "{print \$1}" slavenodes))
for node in ${nodes[*]}
do
    ssh ubuntu@$node "mkdir -p /home/ubuntu/myscript/"
    rsync -r --progress ~/myscript/hosts ubuntu@$node:/home/ubuntu/myscript/hosts
    expect expect_hosts.sh $node ubuntu ubuntu123456
done
```
## 3. diable firewall and seLinux
```shell
#close firewall immediately
systemctl stop firewalld
#disable firewall onboot
systemctl disable firewalld
```
setfirewall_selinux.sh (not working :( ):
```shell
#!/b
for a in {1..8};
do
ssh root@n$a  systemctl stop firewalld
systemctl disable firewalld
setenforce 0
cd /etc/selinux
sed 's/=permissive/=disabled/g' config > config.out
mv -f config.out config
reboot
done
```
## 4.set NTP service
```shell
//config zone infomation
date
#if not CST, do:
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
#comment the default servers and add local ntp server n1
cd /etc
sed 's/server /#server /g' ntp.conf >ntp.conf.out1
sed 's/#server 3.centos.pool.ntp.org iburst/server 10.66.27.61/g' ntp.conf.out1 >ntp.conf.out2
mv -f ntp.conf.out1 ntp.conf
mv -f ntp.conf.out2 ntp.conf
ntpdate -u 10.66.27.61
systemctl restart ntpd
systemctl enable ntpd
hwclock --systohc

#server 0.centos.pool.ntp.org iburstsh  
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
```
## 4.install java
we choose java version 1.8u162 which is the latest version which is tested and recommeded officially.
```shell
#uninstall the installed openjdk
rpm -qa | grep jdk
rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.161-2.b14.el7.x86_64
rpm -e --nodeps java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64
#install out jdk
mkdir /usr/java
tar xvzf /root/Downloads/jdk*.tar.gz -C /usr/java
vi /etc/profile
#add setting
export JAVA_HOME=/usr/java/jdk1.8.0_162
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
:wq
source /etc/profile
java -version
```
## 5.config ISCSI service
CHAP off
```shell
#discover iscsi targets
iscsiadm --mode discoverydb --type sendtargets --portal 10.66.27.43 --discover
#login
iscsiadm -m node –T iqn.2012-06.com::iscsi.datanode4 -p 10.66.27.43 -l
#automatic login on boot
iscsiadm -m node -d 1 -T iqn.2012-06.com::iscsi.datanode4 -p 10.66.27.43:3260 --op update -n node.startup -v automatic
fdisk -l
fdisk /dev/sdb
n
p
<Enter>
.
.
w
#partprobe is a program that informs the operating system kernel of partition #table changes, by requesting that the operating system re-read the partition #table.
partprobe /dev/sdb1
#format partition
mkfs.ext4 /dev/sdb1
#disable automatic check
tune2fs -i 0 -c 0 /dev/sdb1
#make the partition
parted  /dev/sdb
mklabel gpt #set the label
mkpart primary 0% 100% #make one part
print
quit
#format partitioned block
mkfs.ext4 -T largefile /dev/sdb1
#mount
mkdir /opt/iscsi
mount  /dev/sdb1 /opt/iscsi
#enable mount on boot
tune2fs -l /dev/sdb1
echo  "UUID=0409e421-c2b4-4198-8141-86f8ca6e5da0 /opt/iscsi ext4  _netdev  0 0" >> /etc/fstab
```
## 6.Using an Internal Package Repository
using n5(10.66.27.68) as the local repository
```shell
#install Apache HTTP Server
sudo yum install httpd
sudo systemctl start httpd
#Cloudera Manager 6
sudo mkdir -p /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/cm6/6.0.0/redhat7/ -P /var/www/html/cloudera-repos
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/cm6
#CDH 6
sudo mkdir -p /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/cdh6/6.0.0/redhat7/ -P /var/www/html/cloudera-repos
sudo wget --recursive --no-parent --no-host-directories https://archive.cloudera.com/gplextras6/6.0.0/redhat7/ -P /var/www/html/cloudera-repos
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/cdh6
sudo chmod -R ugo+rX /var/www/html/cloudera-repos/gplextras6
```
modify the client configuration to use local repository
```shell
vi /etc/yum.reps.d/cloudera-manager.repo
[cloudera-manager]
name=Cloudera Manager 6.0.0
baseurl=http://10.66.27.68/cloudera-repos/cm6/6.0.0/redhat7/yum/
gpgkey=http://10.66.27.68/cloudera-repos/cm6/6.0.0/redhat7/yum/RPM-GPG-KEY-cloudera
gpgcheck=1
enabled=1
autorefresh=0
type=rpm-md

vi /etc/hosts
#add one line
10.66.27.68 archive.cloudera.com
```
# Install Cloudera Manager Server
## 1.Install Cloudera Manager Packages
```shell
sudo yum install cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server
#optional if you are using Oracle database
# Locate the line that begins with export CMF_JAVA_OPTS and change the -Xmx2G option to -Xmx4G.
```
## 2.Enable Auto-TLS(Not recommanded)
```shell
sudo JAVA_HOME=/usr/java/jdk1.8.0_162 /opt/cloudera/cm-agent/bin/certmanager setup --configure-services
INFO:root:Logging to /var/log/cloudera-scm-agent/certmanager.log
```
## 3.Install and Configure Databases
### install the MySQL database.
```shell
#uninstall obsolete mysql databases
rpm -qa | grep mysql
rpm -e --nodeps <installed mysql databases>
cd /root/Downloads
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum update
yum install mysql-server
systemctl start mysqld
```
### Configure and Starting MySQL Server
```shell
sudo systemctl stop mysqld
#Move old InnoDB log files /var/lib/mysql/ib_logfile0 and /var/lib/mysql/ib_logfile1 out of /var/lib/mysql/ to a backup location.
mkdir /root/Documents/MySQL_InnoDBfile_backup
mv /var/lib/mysql/ib_logfile0 /var/lib/mysql/ib_logfile1 /root/Documents/MySQL_InnoDBfile_backup/
#Update /etc/my.cnf with Cloudera recommended settings:
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
transaction-isolation = READ-COMMITTED
# Disabling symbolic-links is recommended to prevent assorted security risks;
# to do so, uncomment this line:
symbolic-links = 0

key_buffer_size = 32M
max_allowed_packet = 32M
thread_stack = 256K
thread_cache_size = 64
query_cache_limit = 8M
query_cache_size = 64M
query_cache_type = 1

max_connections = 550
#expire_logs_days = 10
#max_binlog_size = 100M

#log_bin should be on a disk with enough free space.
#Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your
#system and chown the specified folder to the mysql user.
log_bin=/var/lib/mysql/mysql_binary_log

#In later versions of MySQL, if you enable the binary log and do not set
#a server_id, MySQL will not start. The server_id must be unique within
#the replicating group.
server_id=1

binlog_format = mixed

read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M

# InnoDB settings
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit  = 2
innodb_log_buffer_size = 64M
innodb_buffer_pool_size = 4G
innodb_thread_concurrency = 8
innodb_flush_method = O_DIRECT
innodb_log_file_size = 512M

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

sql_mode=STRICT_ALL_TABLES

#set MySQL starts at boot:
sudo systemctl enable mysqld
#start MySQL
sudo systemctl start mysqld
#set the MySQL root password(qsmwwy123) and other security-related settings
/usr/bin/mysql_secure_installation
#install JDBC driver
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.46.tar.gz
tar zxvf mysql-connector-java-5.1.46.tar.gz
sudo mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.46
sudo cp mysql-connector-java-5.1.46-bin.jar /usr/share/java/mysql-connector-java.jar
```
Creating Databases for Cloudera Software
```shell
create database scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
create database amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
create database rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
create database metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
create database sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
create database nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
create database navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
create database hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
create database monitor DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
create database oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'qsmwwy123';
GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'qsmwwy123';
GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'qsmwwy123';
GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'qsmwwy123';
GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY 'qsmwwy123';
GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'qsmwwy123';
GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'qsmwwy123';
GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'qsmwwy123';
GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'qsmwwy123';
GRANT ALL ON monitor.* TO 'monitor'@'%' IDENTIFIED BY 'qsmwwy123';
```
### Set up the Cloudera Manager Database
```shell
sudo /opt/cloudera/cm/schema/scm_prepare_database.sh mysql scm scm
```
## 4.Install CDH and Other Software
```shell
sudo systemctl start cloudera-scm-server
#tail the startup process, if needed
sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
```
## 5.Deploy CDH and Agents

<center>
<img src="https://raw.githubusercontent.com/HongyiJiang/cloudera/master/specify%20hosts.JPG"width="960" height="488.5"/>
Figure 1. Specify Hosts
</center>

<center>
<img src="https://raw.githubusercontent.com/HongyiJiang/cloudera/master/Inspect%20Hosts.JPG"width="960" height="488.5"/>
Figure 2. Inspect Hosts for Correctness
</center>

### Fix the warnings and run validations again
```shell
#Fix multiple time zones are used in cluster. 
timedatectl  set-timezone Asia/Shanghai
timedatectl

#Set swappiness to maximum of 10
#configure at runtime
sysctl vm.swappiness=10
#effect at reboot, add vm.swappiness=10
vi /etc/sysctl.conf 

#Disable Transparent Huge Page Compaction
echo never > /sys/kernel/mm/transparent_hugepage/defrag
echo never > /sys/kernel/mm/transparent_hugepage/enabled
vi  /etc/rc.local

#upgrade Psycopg2
yum -y install epel-release
yum -y install python-pip
yum clean all
pip install --upgrade psycopg2
```

### Run the Validation again

<center>
<img src="https://raw.githubusercontent.com/HongyiJiang/cloudera/master/validation.JPG"width="960" height="488.5"/>
Figure 3. Inspect Hosts for Correctness
</center>

### Assign Roles
<center>
<img src="https://raw.githubusercontent.com/HongyiJiang/cloudera/master/roles.JPG" width="960" height="217" />
Figure 4. View Roles by Hosts
</center>

# Problems
## 1.JobHistory Server failed to start ,user hdfs not availble
```shell
#use user hdfs and create dir /tmp and /user
[root@n1 ~]# su hdfs
This account is currently not available.
[root@n1 ~]# cat /etc/passwd | grep hdfs
hdfs:x:378:377:Hadoop HDFS:/var/lib/hadoop-hdfs:/sbin/nologin
[root@n1 hadoop-hdfs]# usermod -s /bin/bash hdfs
[root@n1 hadoop-hdfs]# su hdfs
bash-4.2$ hadoop fs -mkdir /tmp 
bash-4.2$ hadoop fs -mkdir /user
bash-4.2$ hadoop fs -ls /
Found 2 items
drwxr-xr-x   - hdfs supergroup          0 2018-10-15 22:53 /tmp
drwxr-xr-x   - hdfs supergroup          0 2018-10-15 22:54 /user
```
problem solved.
Encount hbase write permission denied which is same problem.
set dfs.permissions: false
Problem solved.
## 2.spark History server failed to start
stderr:
```shell
log4j:ERROR Could not instantiate appender named "redactorForRootLogger".
Exception in thread "main" java.lang.reflect.InvocationTargetException
    at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
    at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
    at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
    at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
    at org.apache.spark.deploy.history.HistoryServer$.main(HistoryServer.scala:278)
    at org.apache.spark.deploy.history.HistoryServer.main(HistoryServer.scala)
Caused by: java.io.FileNotFoundException: Log directory specified does not exist: hdfs://n1:8020/user/spark/applicationHistory
    at org.apache.spark.deploy.history.FsHistoryProvider.org$apache$spark$deploy$history$FsHistoryProvider$$startPolling(FsHistoryProvider.scala:214)
    at org.apache.spark.deploy.history.FsHistoryProvider.initialize(FsHistoryProvider.scala:160)
    at org.apache.spark.deploy.history.FsHistoryProvider.<init>(FsHistoryProvider.scala:156)
    at org.apache.spark.deploy.history.FsHistoryProvider.<init>(FsHistoryProvider.scala:78)
    ... 6 more
Caused by: java.io.FileNotFoundException: File does not exist: hdfs://n1:8020/user/spark/applicationHistory
    at org.apache.hadoop.hdfs.DistributedFileSystem$29.doCall(DistributedFileSystem.java:1498)
    at org.apache.hadoop.hdfs.DistributedFileSystem$29.doCall(DistributedFileSystem.java:1491)
    at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
    at org.apache.hadoop.hdfs.DistributedFileSystem.getFileStatus(DistributedFileSystem.java:1506)
    at org.apache.spark.deploy.history.FsHistoryProvider.org$apache$spark$deploy$history$FsHistoryProvider$$startPolling(FsHistoryProvider.scala:204)
    ... 9 more
```
Try create directory manuelly
```shell
bash-4.2$ hadoop fs -mkdir /user/spark
bash-4.2$ hadoop fs -mkdir /user/spark/applicationHistory
```
Problem solved.
## 3.Impala Daemon failed to start
Invalid short-circuit reads configuration:
  - Impala cannot read or execute the parent directory of dfs.domain.socket.path
Find dfs.domain.socket.path directory in hdfs settings, than create and change the owner of the directory.
```shell
[root@n1 run]# mkdir hdfs-sockets
[root@n1 run]# cd hdfs-sockets/
[root@n1 hdfs-sockets]# mkdir dn
[root@n1 hdfs-sockets]# dir
dn
[root@n1 hdfs-sockets]# ls -l
total 0
drwxr-xr-x 2 root root 40 Oct 15 23:52 dn
[root@n1 hdfs-sockets]# chown impala /var/run/hadoop-hdfs/
hadoop-hdfs-root-nfs3.pid  privileged-root-nfs3.pid   
[root@n1 hdfs-sockets]# chown impala /var/run/hadoop-hdfs/
hadoop-hdfs-root-nfs3.pid  privileged-root-nfs3.pid   
[root@n1 hdfs-sockets]# chown impala /var/run/hdfs-sockets/dn
```
impala started successfully.
Problem solved.
