# SGE-On-CentOS

本工程主要介绍了如何在CentOS上安装并使用SGE（Sun Grid Engine）。使用配合使用NFS的SGE搭建计算集群，可以实现任务多机器并行运行。

#### 工程文件夹说明

softwares                      #安装包及其他软件包

README.md                #安装、测试与SGE操作说明文档（本文件）

## 一. 安装

本次安装

SGE安装包下载地址：  https://arc.liv.ac.uk/downloads/SGE/releases/8.1.9/sge-8.1.9.tar.gz

#### 1. 主控节点安装

命令行执行，修改hostname

```shell
hostnamectl set-hostname qmaster.local
```

命令行执行，修改hosts文件，添加主控节点和两个计算节点信息

```shell
vi /etc/hosts
192.168.98.134 qmaster.local qmaster
192.168.98.135 compute01.local compute01
192.168.98.136 compute02.local compute02
```

命令行执行，创建共享目录

```shell
mkdir -p /BiO/src
```

命令行执行，安装epel源

```shell
yum -y install epel-release
```

命令行执行，安装依赖库

```shell
yum -y install jemalloc-devel openssl-devel ncurses-devel pam-devel libXmu-devel hwloc-devel hwloc hwloc-libs java-devel javacc ant-junit libdb-devel motif-devel csh ksh xterm db4-utils perl-XML-Simple perl-Env xrog-x11-fonts-ISO8859-1-100dpi xrog-x11-fonts-ISO8859-1-75dpi
```

命令行执行，添加sgeadmin用户组，及sgeadmin用户

```shell
groupadd -g 490 sgeadmin
useradd -u 495 -g 490 -m -d /home/sgeadmin -s /bin/bash -c "SGE Admin" sgeadmin
```

命令行执行，修改sudo文件，添加一行配置

```shell
visudo
%sgeadmin ALL=(ALL) NOPASSWD: ALL
```

命令行执行，下载并编译SGE。（如果您下载不了SGE安装包，请到本工程/softwares目录下获取）

```shell
cd /BiO/src
wget https://arc.liv.ac.uk/downloads/SGE/releases/8.1.9/sge-8.1.9.tar.gz
tar -zxvf sge-8.1.9.tar.gz
cd sge-8.1.9/source/
sh scripts/bootstrap.sh && /.aimk && /.aimk -man
export SGE_ROOT=/Bio/gridengine && mkdir $SGE_ROOT
echo Y | ./scripts/distinst -local -allall -libs -noexit
chown -R sgeadmin.ageadmin /BiO/gridengine
```

命令行执行，安装SGE qmaster节点
```shell
cd $SGE_ROOT
./install_qmaster
```

安装过程中，需要同意一些默认配置。
> press enter at the intro screen
 ress "y" and then specify sgeadmin as the user id
 eave the install dir as /BiO/gridengine
 ou will now be asked about port configuration for the master, normally you would choose the default (2) which uses the /etc/services file
 ccept the sge_qmaster info
 ou will now be asked about port configuration for the master, normally you would choose the default (2) which uses the /etc/services file
 ccept the sge_execd info
 eave the cell name as "default"
 nter an appropriate cluster name when requested
 eave the spool dir as is
 ress "n" for no windows hosts!
 ress "y" (permissions are set correctly)
 ress "y" for all hosts in one domain
 f you have Java available on your Qmaster and wish to use SGE Inspect or SDM then enable the JMX MBean server and provide the requested information - probably answer "n" at this point!
 ress enter to accept the directory creation notification
 nter "classic" for classic spooling (berkeleydb may be more appropriate for large clusters)
 ress enter to accept the next notice
 nter "20000-20100" as the GID range (increase this range if you have execution nodes capable of running more than 100 concurrent jobs)
 ccept the default spool dir or specify a different folder (for example if you wish to use a shared or local folder outside of SGE_ROOT
 nter an email address that will be sent problem reports
 ress "n" to refuse to change the parameters you have just configured
 ress enter to accept the next notice
 ress "y" to install the startup scripts
 ress enter twice to confirm the following messages
 ress "n" for a file with a list of hosts
 nter the names of your hosts who will be able to administer and submit jobs (enter alone to finish adding hosts)
 kip shadow hosts for now (press "n")
 hoose "1" for normal configuration and agree with "y"
 ress enter to accept the next message and "n" to refuse to see the previous screen again and then finally enter to exit the installer

命令行执行，安装NFS，将主控节点目录共享

```shell
yum -y install nfs-utils
vi /etc/exports
/BiO 192.168.98.134/24(rw,no_root_squash)
```

命令行执行，
```shell
systemctl start rpcbind nfs-server
systemctl enable rpcbind nfs-server
```
至此qmaster主控节点安装配置完毕！

#### 2. 计算节点安装（以compute01为例）

命令行执行，安装依赖库

```shell
yum -y install hwloc-devel
```

命令行执行，修改hostname

```shell
hostnamectl set-hostname compute01.local
```

命令行执行，修改hosts文件，添加主控节点和两个计算节点信息

```shell
vi /etc/hosts
192.168.98.134 qmaster.local qmaster
192.168.98.135 compute01.local compute01
192.168.98.136 compute02.local compute02
```

groupadd -g 490 sgeadmin
useradd -u 495 -g 490 -r -m -d /home/sgeadmin -s /bin/bash -c "SGE Admin" sgeadmin

yum -y install nfs-utils
systemctl start rpcbind
systemctl enable rpcbind

mkdir /BiO
mount -t nfs 192.168.56.101:/BiO /BiO
vi /etc/fstab
192.168.98.134:/BiO /BiO nfs defaults 0 0

export SGE_ROOT=/BiO/gridengine
export SGE_CELL=default
cd $SGE_ROOT
./install_execd
cp /BiO/gridengine/default/common/settings.sh /etc/profile.d/


## 二. 测试


## 三. SGE操作