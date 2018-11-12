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

> press "y" and then specify sgeadmin as the user id

> leave the install dir as /BiO/gridengine

> You will now be asked about port configuration for the master, normally you would choose the default (2) which uses the /etc/services file

> Accept the sge_qmaster info

> You will now be asked about port configuration for the master, normally you would choose the default (2) which uses the /etc/services file

> Accept the sge_execd info

> Leave the cell name as "default"

> Enter an appropriate cluster name when requested

> Leave the spool dir as is

> Press "n" for no windows hosts!

> Press "y" (permissions are set correctly)

> Press "y" for all hosts in one domain

> If you have Java available on your Qmaster and wish to use SGE Inspect or SDM then enable the JMX MBean server and provide the requested information - probably answer "n" at this point!

> Press enter to accept the directory creation notification

> E nter "classic" for classic spooling (berkeleydb may be more appropriate for large clusters)

> Press enter to accept the next notice

> Enter "20000-20100" as the GID range (increase this range if you have execution nodes capable of running more than 100 concurrent jobs)

> Accept the default spool dir or specify a different folder (for example if you wish to use a shared or local folder outside of SGE_ROOT

> Enter an email address that will be sent problem reports

> Press "n" to refuse to change the parameters you have just configured

> Press enter to accept the next notice

> Press "y" to install the startup scripts

> Press enter twice to confirm the following messages

> Press "n" for a file with a list of hosts

> Enter the names of your hosts who will be able to administer and submit jobs (enter alone to finish adding hosts)

> Skip shadow hosts for now (press "n")

> Choose "1" for normal configuration and agree with "y"

> Press enter to accept the next message and "n" to refuse to see the previous screen again and then finally enter to exit the installer

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

命令行执行，添加sgeadmin用户组，添加sgeadmin用户

```shell
groupadd -g 490 sgeadmin
useradd -u 495 -g 490 -r -m -d /home/sgeadmin -s /bin/bash -c "SGE Admin" sgeadmin
```

命令行执行，安装NFS，并启动服务

```shell
yum -y install nfs-utils
systemctl start rpcbind
systemctl enable rpcbind
```

命令行执行，创建共享目录，将主控节点目录共享至计算节点

```shell
mkdir /BiO
mount -t nfs 192.168.56.101:/BiO /BiO
vi /etc/fstab
192.168.98.134:/BiO /BiO nfs defaults 0 0
```

命令行执行，安装计算节点

```shell
export SGE_ROOT=/BiO/gridengine
export SGE_CELL=default
cd $SGE_ROOT
./install_execd
cp /BiO/gridengine/default/common/settings.sh /etc/profile.d/
```

## 二. 测试

命令行执行，在qmaster或两台compute01、compute02均可执行qhost，查看集群主机列表

```shell
qhost
HOSTNAME        ARCH       NCPU NSOC NCOR NTHR  LOAD  MEMOUT  MEMUSE  SWAPTO   SWPUS
------------------------------------------------------------------------------------
global          -             -    -    -    -     -       -       -       -       -
compute01       lx-amd64     32    2   16   32  2.07  377.7G   42.4G    4.0G   82.5M
compute02       lx-amd64     32    2   16   32  0.04  377.7G    2.8G    4.0G   77.2M
```

可以通过提交一个简单的任务（job），测试SGE的功能。

这个任务是打印当前执行及其的Linux内核版本号。在任意一个计算节点（如compute01），命令行执行，vi编辑任务执行脚本

```shell
vi name.sge
#!/bin/bash
uname -a
```

命令行执行，使用qsub提交任务并执行

```shell
qsub uname.age
You job 1("uname.age") has been submitted
```

执行完毕后，您可以在执行任务的路径下看到任务运行得到的日志输出（包括错误日志）

```shell
ls
uname.sge.e1  uname.sge.o1
cat uname.sge.o1
Linux compute01.local 3.10.0-862.el7.x86_64 #1 SMP Fri Apr 20 16:44:24 UTC 2018 x86_64 x86_64 GUN/Linux
```

## 三. SGE操作

qhost  查看集群主机列表

qstat  查看集群任务执行状态（可使用watch -n 10 -d qstat 持查看执行状态）

qdel [jobid] 通过jobid删除任务

qsub  提交任务
  qsub -cwd ./run.sh
其中-cwd表示以当前目录作为执行目录，否则命令将以sgeadmin用户目录作为当前目录。