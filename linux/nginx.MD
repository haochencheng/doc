##  nginx反向代理
https://segmentfault.com/a/1190000018109309

### nginx编译安装和一键安装脚本

####    编译安装nginx
1.安装依赖包

```
yum -y install openssl openssl-devel zlibzlib-devel gcc gcc-c++
```

2.安装pcre

安装nginx所需要的pcre库（为了使nginx支持http rewrite模块）

```
mkdir -p/home/darren/tools
cd /home/darren/tools/
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.37.tar.gz
tar zxf pcre-8.37.tar.gz
cd pcre-8.37
./configure
make && make install
cd ..
```

如果安装完以后需要重新安装pcre，则需要

```
rm -rf/usr/local/share/man/man3/pcre*
```


3.安装nginx

创建nginx用户


```
/usr/sbin/groupadd -f nginx                     
/usr/sbin/useradd -g nginx nginx

cd /home/darren/tools/
wget http://nginx.org/download/nginx-1.8.1.tar.gz
wgethttp://nginx.org/download/nginx-1.2.6.tar.gz
tarzxf nginx-1.8.1.tar.gz
cd nginx-1.8.1
./configure --user=nginx --group=nginx --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
make && makeinstall
```

 

检查nginx安装


```
/usr/local/nginx/sbin/nginx -v
#提示错误：/usr/local/nginx/sbin/nginx: error while loading shared libraries:libpcre.so.1: cannot open shared object file: No such file ordirectory
yum install pcre -y 则不会出现这种错误
find /-name libpcre.so.1
/usr/local/lib/libpcre.so.1
echo '/usr/local/lib/'>>/etc/ld.so.conf
ldconfig
```





检查语法


```
/usr/local/nginx/sbin/nginx -t
```


启动nginx服务


```
/usr/local/nginx/sbin/nginx
netstat -lnt
```

浏览器中输入10.1.1.1测试





####    附一个一键安装脚本：


```
#!/bin/sh
#for one install nginx
#for wangjiadongge
 
#install centos source
rpm -Uvh http://mirrors.yun-idc.com/epel/6Server/x86_64/epel-release-6-8.noarch.rpm
 
#install plugin
yum install gcc gcc-c++ openssl-devel pcre-devel zlib-devel -y
 
#create user and group
groupadd nginx 
useradd nginx -g nginx -s /sbin/nologin -M
 
#cd /tmp/
[ -d /tmp/download ] || mkdir -p /tmp/download
cd /tmp/download/
 
 
#wget nginx from www.nginx.org
wget http://nginx.org/download/nginx-1.8.1.tar.gz 
tar zxvf nginx-1.8.1.tar.gz
cd nginx-1.8.1
./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_stub_status_module --with-http_ssl_module
make && make install

chown -R nginx.nginx /usr/local/nginx/
echo "/usr/local/nginx/sbin/nginx" >>/etc/rc.local 
#end
```



#nginx负载均衡脚本

```
mv /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx.conf.bak
echo "
user  nginx;
worker_processes  4;
events {
    use epoll;
    worker_connections  10240;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;
    sendfile        on;  
    keepalive_timeout  120;
    include conf.d/*.conf;
}

" >>/usr/local/nginx/conf/nginx.conf


mkdir /usr/local/nginx/conf/conf.d
echo "
server {
        listen       80;
        server_name  login.xiaodongge.com;
        access_log   logs/host.access.log  main;

        location / {
            proxy_pass http://loginserver;
        }
}

upstream loginserver {
        server 10.86.10.21:8300;
        server 10.86.10.22:8300;
        server 10.86.10.23:8300;
    }
```

解释：nginx做负载均衡默认是轮询负载，下面逐一来看：

#####   默认的负载方式，轮询

```
upstream myapp1 {
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }
    server {
        listen 80;
        location / {
            proxy_pass http://myapp1;
        }
    }
```

####    最小连接数

哪台服务器连接的最少就向哪台服务器上负载

```
upstream myapp1 {
        least_conn;
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }
```

####  ip hash

如果一个连接建立，就会始终和这台服务器连接

```
upstream myapp1 {
    ip_hash;
    server srv1.example.com;
    server srv2.example.com;
    server srv3.example.com;
}
```

####   权重，自定义设置哪台服务器负载的多

```
upstream myapp1 {
        server srv1.example.com weight=3;
        server srv2.example.com;
        server srv3.example.com;
    }
```



### nginx自动启动脚本

```
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/etc/nginx/nginx.conf"

[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx

lockfile=/var/lock/subsys/nginx

make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:.*--user=" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -n "$user" ]; then
      if [ -z "`grep $user /etc/passwd`" ]; then
         useradd -M -s /bin/nologin $user
      fi
      options=`$nginx -V 2>&1 | grep 'configure arguments:'`
      for opt in $options; do
          if [ `echo $opt | grep '.*-temp-path'` ]; then
              value=`echo $opt | cut -d "=" -f 2`
              if [ ! -d "$value" ]; then
                  # echo "creating" $value
                  mkdir -p $value && chown -R $user $value
              fi
          fi
       done
    fi
}

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

保存脚本文件后设置文件的执行权限：


```
chmod a+x /etc/init.d/nginx
```

然后，就可以通过该脚本对nginx服务进行管理了：


```
/etc/init.d/nginx start
/etc/init.d/nginx stop
```

使用chkconfig进行管理

上面的方法完成了用脚本管理nginx服务的功能，但是还是不太方便，比如要设置nginx开机启动等。这时可以使用chkconfig来设置。

先将nginx服务加入chkconfig管理列表：


```
chkconfig --add /etc/init.d/nginx
```

加完这个之后，就可以使用service对nginx进行启动，重启等操作了。

```
service nginx start
service nginx stop
```

设置终端模式开机启动：

```
chkconfig nginx on
```

/opt/gateway-admin/ucommune-gateway-admin/dist/


### 启动nginx

```

/usr/local/nginx/sbin/nginx -t
检查配置文件nginx.conf的正确性
启动Nginx
sudo /usr/local/nginx/sbin/nginx

/usr/local/nginx/sbin/nginx -t
停止Nginx

/usr/local/nginx/sbin/nginx -s stop
重新加载配置文件

/usr/local/nginx/sbin/nginx -s reload
```

 

```
listen 9527;                                                                                                       
        server_name gateway-admin;                                                                                         
        location / {                                                                                                       
           alias  /opt/gateway-admin/ucommune-gateway-admin/dist/;                                                         
           index index.html;                                                                                               
        } 
```

