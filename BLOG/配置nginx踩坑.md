# 在服务器CentOS7上部署nginx遇到的问题

安装的是nginx1.14.0版本

## 下载源码

```shell
wget http://nginx.org/download/nginx-1.14.tar.gz
tar -xzf nginx-1.14.0.tar.gz
cd nginx-1.14.0
```

## 安装编译环境

```shell
yum update
yum -y install gcc pcre pcre-devel zlibzlib-devel openssl openssl-devel
```

## 编译安装

```shell
#添加用户和组
groupadd grp0
useradd -g grp0 usr0
#配置
./configure \
--user=usr0 \
--group=grp0 \
--prefix=/usr/local/nginx \
--with-http_ssl_module \
--with-http_stub_status_module \
--with-http_realip_module \
--with-threads
#编译安装
make && make install
```

## 验证

```shell
/usr/local/nginx/sbin/nginx -V
```

输出：

```shell
nginx version: nginx/1.14.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/usr/local/nginx --with-http_ssl_module --with-http_stub_status_module --with-threads
```

## 创建软链接

```shell
ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx
```

## 开机自启动

```shell
vim /etc/init.d/nginx
```

输入如下内容

```shell
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /usr/local/nginx/conf/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /usr/local/nginx/logs/nginx.pid
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)
NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"
[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
lockfile=/var/lock/subsys/nginx
make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
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

赋予脚本可执行权限

```shell
chmod a+x /etc/init.d/nginx
```

将nginx服务加入chkconfig管理列表

```shell
chkconfig --add /etc/init.d/nginx
chkconfig nginx on
# 启动
systemctl start nginx
```

如果提示错误信息

```shell
Can't open PID file /var/run/nginx.pid (yet?) after start: No such file or directory
```

将上面脚本这一句

```shell
# pidfile:   /usr/local/nginx/logs/nginx.pid
```

前面的#去掉，即取消注释

```shell
pidfile:     /usr/local/nginx/logs/nginx.pid
```

问题解决

## 测试

```shell
curl localhost:80
```

输出

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## 配置优化

下列配置在nginx.conf里与http节点同级

```shell
cd /usr/local/nginx/conf
vim nginx.conf
```

```shell
user  www www;
worker_processes auto;
worker_rlimit_nofile 51200;

events
    {
        use epoll;
        worker_connections 51200;
        multi_accept on;
    }
```

http节点下加入下列配置，将会从指定目录加载配置文件

```shell
  include       mime.types;
  include /etc/nginx/*.conf; #添加了这一行
  default_type  application/octet-stream;
```

## 常用命令

```shell
# 启动
systemctl start nginx
# 查看状态
systemctl status nginx
# 停止
systemctl stop nginx

# 重载配置
nginx -s reload
# 测试配置是否正确
nginx -t
```

## 参考资料

<https://www.cnblogs.com/stulzq/p/9291223.html>
