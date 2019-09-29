---
title: centOS7常用命令
comments: false
date: 2019-09-03 10:06:22
categories: centOS7
tags: centOS7
---

下文命令中的中括号[ ]，代表可选的相关命令或服务名。
> 如：lsof -i:[端口号]  => lsof -i:8080

### 文件操作的相关命令

| 命令                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| cd /opt/                                                     | 进入目录                                                     |
| ls                                                           | 列出当前目录下的所有文件名                                   |
| ls -l （ll）                                                 | 列出当前目录下的所有非隐含文件                               |
| ll -a                                                        | 查看隐含文件                                                 |
| qwd                                                          | 查看当前目录                                                 |
| mkdir 目录1 目录2                                            | 创建平级目录                                                 |
| mkdir -p parent/child/grandson                               | 创建父子目录                                                 |
| mv 源文件或目录 目标文件或目录                               | 移动文件（也可以重命名）                                     |
| cp -R 文件名 目标目录                                        | 复制文件                                                     |
| rm -f 文件名 (慎用 rm -f /*,会删除根目录下所有文件)          | 删除文件                                                     |
| rm -rf parent 目录名                                         | 删除目录                                                     |
| vim /etc/profile                                             | 打开Profile文件配置环境变量（按i编辑模式；按Esc退出编辑；输入"：wq!"保存退出） |
| source /etc/profile                                          | 使配置的环境变量生效                                         |
| cat /logs/catalina.out                                       | 查看文件内容                                                 |
| tar -xzvf file.tar.gz                                        | 解压tar.gz                                                   |
| tar -xvf file.tar                                            | 解压.tar包                                                   |
| tar -xjvf file.tar.bz2                                       | 解压.tar.bz2                                                 |
| unrar e file.rar                                             | 解压.rar                                                     |
| unzip                                                        | 解压.zip                                                     |
| tar –cvf jpg.tar *.jpg                                       | 将目录里所有jpg文件打包成tar.jpg                             |
| tar –czf jpg.tar.gz *.jpg                                    | 将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz |
| tar –cjf jpg.tar.bz2 *.jpg                                   | 将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2 |
| tar –cZf jpg.tar.Z *.jpg                                     | 将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z |
| tar 相关命令描述                                             |                                                              |
| -c                                                           | 建立压缩档案                                                 |
| -x                                                           | 解压                                                         |
| -t                                                           | 查看内容                                                     |
| -r                                                           | 向压缩归档文件末尾追加文件                                   |
| -u                                                           | 更新原压缩包中的文件                                         |
| 以下五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。下面的参数是根据需要在压缩或解压档案时可选的。 |                                                              |
| -z                                                           | 有gzip属性的                                                 |
| -j                                                           | 有bz2属性的                                                  |
| -Z                                                           | 有compress属性的                                             |
| -v                                                           | 显示所有过程                                                 |
| -O                                                           | 将文件解开到标准输出                                         |


### systemctl 
- 注册在系统中的标准化程序，统一管理方式：
  - systemctl start 服务名(xxxx.service)
  - systemctl restart 服务名(xxxx.service)
  - systemctl stop 服务名(xxxx.service)
  - systemctl reload 服务名(xxxx.service)
  - systemctl status 服务名(xxxx.service)
- 查看服务的方法 /usr/lib/systemd/system
- 查看服务的命令
  - systemctl list-unit-files
  - systemctl --type service
- 通过systemctl 命令设置自启动
- 自启动 systemctl enable service_name
- 不自启动 systemctl disable service_name

### 查看后台进程的三种方式
1. ps -ef|grep [服务名]|grep -v grep
2. netstat -anp|grep [端口号]
3. lsof -i:61616

*杀死进程：* kill -9 [pid]


### 防火墙相关
| 命令                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| reboot                                                       | 重启服务器                                                   |
| clear                                                        | 清屏                                                         |
| systemctl start firewalld                                    | 打开防火墙                                                   |
| systemctl stop firewalld                                     | 关闭防火墙                                                   |
| firewall-cmd --reload 或者 systemctl reload firewalld        | 重启防火墙                                                   |
| firewall-cmd --state 或者 systemctl status firewalld         | 查看防火墙状态                                               |
| systemctl enable firewalld                                   | 开机自启动防火墙                                             |
| systemctl disable firewalld                                  | 禁止开机启动防火墙                                           |
| firewall-cmd --list-ports                                    | 查看已打开的端口                                             |
| firewall-cmd --permanent --zone=public  --add-port=8080/tcp  | 开放指定端口，其中 permanent 表示永久生效，public 表示作用域，8080/tcp表示端口和类型 |
| firewall-cmd --permanent --zone=public --remove-port=8080/tcp | 关闭端口                                                     |
| vi /etc/sysconfig/iptables                                   | 编辑防火墙配置文件                                           |


### 其他
| 命令                      | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| su root                   | 切换到root用户                                               |
| exit                      | 回到用户权限                                                 |
| reboot                    | 重启                                                         |
| clear                     | 清屏                                                         |
| ifconfig                  | 查看centos ip                                                |
| /etc/init.d/sshd 〔可选〕 | 查看ssh服务用法〔start/stop/restart/reload/condrestart/status〕 |
| cat /etc/sysconfig/i18n   | 查看系统编码方式                                             |
| yum -y install lrzsz      | 安装在线导入安装包的插件                                     |


### 图形界面和命令行界面切换
#### 1. 临时切换（通用）
| 命令   | 说明                   |
| ------ | ---------------------- |
| init 3 | 图形界面切换命令行界面 |
| init 5 | 命令行界面切换图形界面 |

#### 2. 永久切换
centOS7
> 1. 首先删除已经存在的符号链接：rm /etc/systemd/system/default.target
> 2. 默认级别转换为3(文本模式)：ln -sf /lib/systemd/system/multi-user.target /etc/systemd/system/default.target 
> 2. 或者默认级别转换为5(图形模式)：ln -sf /lib/systemd/system/graphical.target /etc/systemd/system/default.target
> 3. 重启：reboot

centOS7以下版本
> - 以管理员权限编辑 /etc/inittab 文件
> - 将 id:5:initdefault: 修改为 id:3:initdefault:
> - 5表示图形化界面，设置为5后表示每次启动系统进入的是图形化界面
> - 3表示命令行界面，设置为3后表示每次启动系统进入的是命令行界面
> - 保存重启即可

### JDK安装及环境变量
查看系统自带的jdk：rpm -qa|grep java
卸载系统自带的jdk：rpm -e --nodeps jdk名
```shell
# 注意：windows下用分号(;)，Linux下用冒号(:)分隔。
export JAVA_HOME=/usr/local/java/jdk1.8.0_11
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```

