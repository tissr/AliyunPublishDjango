# AliyunPublishDjango
阿里云 部署django全攻略

## 1.登录root用户在系统下新建用户
```
useradd -m zhaozhao
```
## 2. 为新用户(`zhaozhao`)添加密码(默认创建的用户没有密码)
```
passwd zhaozhao
```
## 3.为新用户添加sudo权限
```
usermod -a -G adm zhaozhao
usermod -a -G sudo zhaozhao
```
## 4.退出root用户,用新账户重新登录Linux
```
exit
ssh zhaozhao@主机ip地址 
```
## 5.在用户根目录下新建项目目录(如果登录后控制键乱码,输入bash,然后回车即可!)

```
mkdir ~/data
mkdir ~/data/{backup,code,logs,server,soft,virtual}
```
> 支持中文`sudo locale-gen zh_CN.UTF-8`

## 6.安装python环境(大多数ubuntu自带,python环境,无需安装),本步骤可跳过
```
sudo apt install python3
```
---
## 为当前用户添加远程认证(可选)
```
ssh-keygen -t rsa # 生成加密算法为 rsa的秘钥

ssh-copy-id zhaozhao@远程ip #将公钥拷贝到服务器端(公钥可多次使用,私钥相当于一卡通!)

```
---

## 7.将本地项目压缩,并通过scp, 传输到`~/data/code`

```
tar -zcvf fangyuanxiaozhan.tar.gz fangyuanxiaozhan
``` 
```
scp fangyuanxiaozhan.tar.gz zhaozhao@远程ip:~/data/code/fangyuanxiaozhan.tar.gz
```
## 8.将本地配置导出到文件,并将文件传输到服务端
```
pip freeze > requirements.txt
```

```
scp requirements.txt zhaozhao@远程ip:~/data/soft/requirements.txt
```

## 9.安装虚拟环境软件,并将virtualenvwrapper.sh配置到shell环境中

```
sudo apt install python-pip
sudo pip install virtualenv
sudo pip install virtualenvwrapper
```

> ![virtualenvwrapper.sh](http://upload-images.jianshu.io/upload_images/3203841-c950305ca0c3b922.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



```
# 添加pytho虚拟环境配置
vim ~/.bashrc
export WORKON_HOME=$HOME/.virtualenvs
source /usr/local/bin/virtualenvwrapper.sh
```
## 10. 创建虚拟环境,安装依赖包
```
# 创建虚拟环境
mkvirtualenv dj_py3 --python="/usr/bin/python3"

# 安装依赖包
pip install -r ~/data/soft/requirements.txt
```

---
> 出现错误:Python.h: No such file or directory
解决方式:`sudo apt-get install python3-dev`

---
##  11.安装Nginx
#### 1.上传软件包到指定目录
 ```
 scp nginx-1.10.3.tar.gz zhaozhao@远程ip:~/data/soft/nginx-1.10.3.tar.gz
 
 scp openssl-1.0.2l.tar.gz zhaozhao@远程ip:~/data/soft/openssl-1.0.2l.tar.gz
 
 scp zlib-1.2.11.tar.gz zhaozhao@远程ip:~/data/soft/zlib-1.2.11.tar.gz
 ```
#### 2.安装pcre(nginx正则匹配依赖)
```
sudo apt-get install libpcre3 libpcre3-dev
```
#### 3.安装zlib
```
tar -zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make
sudo make install
cd ..
```
#### 4.安装openssl(解压即可,目录`~/data/soft/openssl-1.0.2l`)
```
tar -zxvf openssl-1.0.2l.tar.gz
```

#### 5.安装nginx

```
#  解压nginx
tar -zxvf nginx-1.10.3.tar.gz
#  进入nginx安装目录
cd nginx-1.10.3

# 在指定位置安装nginx
./configure --prefix=/opt/nginx/ --with-openssl=~/data/soft/openssl-1.0.2l

# 编译
make
# 安装
sudo make install
# 启动
cd /opt/nginx/sbin
sudo ./nginx
# 查看
ps ajx | grep nginx
```
## 12.安装mysql(一定要设置密码,会避免很多麻烦)
```
sudo apt-get install mysql-server
```

## 13.配置uwsgi


> 1. 在项目目录的同名模块下,新建配置文件`uwsgi.ini`

> ![uwsgi.ini](http://upload-images.jianshu.io/upload_images/3203841-73051ea1df0e9700.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> 2.在配置文件中加入以下内容

```
[uwsgi]
# 配置nginx
socket = 127.0.0.1:3309
# 配置项目目录
chdir = /home/zhaozhao/data/code/fangyuanxiaozhan
# 配置入口模块
wsgi-file = fangyuanxiaozhan/wsgi.py
# 开启master, 将会多开一个管理进程, 管理其他服务进程
master = True
# 服务器开启的进程数量
processes = 2
# 服务器进程开启的线程数量
threads = 4
# 以守护进程方式提供服, 输出信息将会打印到log中
# daemonize = wsgi.log
# 退出的时候清空环境变量
vacuum = true
# 进程pid
pidfile=uwsgi.pid
```

> 3.以配置好的文件 uwsgi.ini启动uwsgi

```
uwsgi --ini uwsgi.ini
```
## 14.配置启动nginx的文件

![nginx.conf](http://upload-images.jianshu.io/upload_images/3203841-13850099037b130a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



> 1. `nginx.conf`配置内容

```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        #location / {
        #    root   html;
        #    index  index.html index.htm;
        #}

    location / {
            # 将nginx所有请求转到uwsgi
            include uwsgi_params;
            # uwsgi的ip与端口
            uwsgi_pass 127.0.0.1:3309;
    }

    location /static {
            alias /home/zhaozhao/data/code/fangyuanxiaozhan/static;
    }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

> 2.重启`nginx`

```
cd /opt/nginx/sbin
sudo ./nginx -s stop
sudo ./nginx 
```
## 重启服务
```
ps ajx | grep uwsgi
kill -9 2844
```

# 部署完成

> 教程涉及到的资源我都通过百度网盘分享给大家,为了便于大家的下载,资源整合到了一张独立的帖子里,链接如下:
http://www.jianshu.com/p/4f28e1ae08b1

