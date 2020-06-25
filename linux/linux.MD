## 链接

####    软链接：

1.软链接，以路径的形式存在。类似于Windows操作系统中的快捷方式

2.软链接可以 跨文件系统 ，硬链接不可以

3.软链接可以对一个不存在的文件名进行链接

4.软链接可以对目录进行链接
####    硬链接:

1.硬链接，以文件副本的形式存在。但不占用实际空间。

2.不允许给目录创建硬链接

3.硬链接只有在同一个文件系统中才能创建

第一，ln命令会保持每一处链接文件的同步性，也就是说，不论你改动了哪一处，其它的文件都会发生相同的变化；

第二，ln的链接又分软链接和硬链接两种，软链接就是ln –s 源文件 目标文件，它只会在你选定的位置上生成一个文件的镜像，不会占用磁盘空间，硬链接 ln 源文件 目标文件，没有参数-s， 它会在你选定的位置上生成一个和源文件大小相同的文件，无论是软链接还是硬链接，文件都保持同步变化。

ln指令用在链接文件或目录，如同时指定两个以上的文件或目录，且最后的目的地是一个已经存在的目录，则会把前面指定的所有文件或目录复制到该目录中。若同时指定多个文件或目录，且最后的目的地并非是一个已存在的目录，则会出现错误信息。

### 查看端口占用
ps -aux | grep tomcat 


### 获取程序进程

```
process_id=$(ps -ef | grep eureka-server | grep -v "grep" | awk '{print $2}')
```

```
#! /bin/sh
yum install zip -y
yum install unzip -y
yum install git -y

curl -s "https://get.sdkman.io" | bash
source "/root/.sdkman/bin/sdkman-init.sh"
sdk install java 8.0.191-oracle -y
sdk install maven 
echo export JAVA_HOME=/root/.sdkman/candidates/java/current >> /etc/profile.d/development.sh
source /etc/profile
ln -s /root/.sdkman/candidates/maven/current /opt/maven

cd /opt
git clone https://github.com/alibaba/nacos.git
cd nacos
git checkout 0.4.0
git checkout -b v0.4.0-release


mvn -Prelease-nacos clean install -U -DskipTests=true

```


### ssh免密登录

```
#!/bin/bash
user=root
pass='123'
ip='192.168.1.154'
expect -c "
    spawn ssh $user@$ip
    expect {
        \"(yes/no)\" {send \"yes\r\"; exp_continue}
        \"password:\" {send \"$pass\r\"; exp_continue}
        \"root@*\" {send \"df -h\r exit\r\"; exp_continue}
    }"
```
