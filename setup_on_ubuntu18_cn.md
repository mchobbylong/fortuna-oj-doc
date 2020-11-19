# Fortuna OJ 部署指南

以下指南将在**纯净的 Ubuntu 18.04.1 LTS 操作系统**上部署 fortuna-oj。

*PS：fortuna-oj 已~~凄惨~~闭源。本文档留给维护者参照使用。*

## 使用自动化脚本（推荐）

使用脚本后无需再进行下述手动部署操作。

请根据脚本提示输入，完成配置。

```sh
wget https://raw.githubusercontent.com/roastduck/fortuna-oj/master/scripts/install.py && sudo python3 install.py
```

## 配置环境

1. [更换 apt 源为清华 Tuna（可选）](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

2. 安装 Git

   ```sh
   sudo apt install -y git
   ```

3. 安装 nginx

   ```sh
   sudo apt install -y nginx
   ```

4. 安装 MariaDB

   ```sh
   sudo apt install -y mariadb-server
   ```

5. 安装 redis

   ```sh
   sudo apt install -y redis
   ```

6. 安装 php 7.2 以及部分 php 插件

   ```sh
   sudo apt install -y software-properties-common apt-transport-https lsb-release ca-certificates
   sudo add-apt-repository ppa:ondrej/php
   sudo apt update
   sudo apt install -y php7.2-fpm php7.2-mysql php7.2-curl php7.2-gd php7.2-mbstring php7.2-xml php7.2-xmlrpc php7.2-zip php7.2-opcache php-redis
   sudo service php7.2-fpm restart
   ```

7. 配置 redis 监听 Unix Socket

   ```sh
   # 以 root 权限编辑
   # /etc/redis/redis.conf
   # 找到以下两行，去除注释符号并作修改；或直接添加
   # 
   # unixsocket /var/run/redis/redis-server.sock
   # unixsocketperm 770
   
   sudo usermod -aG redis www-data
   sudo service redis-server restart
   ```

8. 配置 MariaDB 最大并发连接数

   ```sh
   # 以 root 权限编辑
   # /etc/mysql/mariadb.conf.d/50-server.cnf
   # 找到 max_connections 行，去除注释符号并作修改
   max_connections = 32768

   sudo service mariadb restart
   ```

9. 重启操作系统以应用变更（用户组变更）

## 部署 fortuna-oj

1. 从 GitHub 拉取 fortuna-oj

   ```sh
   cd /var/www
   sudo mkdir foj
   sudo chown www-data:www-data foj
   sudo -u www-data git clone -b master https://github.com/roastduck/fortuna-oj foj
   ```

2. 导入相关数据库

   ```sh
   # 在导入的 sql 文件中默认新建一个数据库用户：
   # 用户名 foj
   # 密码 foj
   # 以及一个 fortuna-oj 专用数据库 foj
   # 请按需修改
   
   sudo mysql < /var/www/foj/migrate/full.sql
   ```

3. 根据第 2 步的数据库相关信息，修改 fortuna-oj 配置文件

   ```sh
   cd /var/www/foj/overriding_config
   sudo -u www-data cp local.php.example local.php
   sudo -u www-data ./gen_secret.py
   # 脚本中输入第 2 步设置的数据库用户名、密码
   # 脚本提示被破环，请无视提示
   ```

   ```php
   # 以 www-data 权限编辑
   # /var/www/foj/overriding_config/local.php
   
   $assign_to_config['cookie_path'] = '/foj';
   $assign_to_config['oj_name'] = 'foj';

   # 此处根据第 2 步设置的数据库名修改
   $db['default']['database'] = 'foj';
   ```

4. 配置服务和自启动 Daemon

   ```sh
   cd /var/www/foj/application
   sudo -u www-data cp daemon.php.example daemon.php

   # 以 www-data 权限编辑
   # /var/www/foj/application/daemon.php
   # 替换其中的 {{db_user}}, {{db_pwd}}, {{db_name}}
   # 为第 2 步的对应设置
   ```

   ```sh
   sudo crontab -e -u www-data
   
   # 编辑器中插入
   */1 * * * * curl http://127.0.0.1/foj/misc/daemon > null
   ```

5. 配置 nginx 网站配置

   ```sh
   sudo rm /etc/nginx/sites-enabled/default
   ```

   ```nginx
   # 以 root 权限创建并编辑
   # /etc/nginx/sites-enabled/foj.conf
   
   server {
       listen 80;
       listen [::]:80;
       
       # server_name foj.com;	# 替换成自己的域名，也可以不填
       
       root /var/www;

       index index.html index.php;
       
       location ~* \.(ico|css|js|gif|jpe?g|png)(\?[0-9]+)?$ {
           expires max;
           log_not_found off;
       }

       location / {
           if ($request_uri = "/") {
               return 302 /foj;
           }
           return 403;
       }
       
       location /foj {
           try_files $uri $uri/ /foj/index.php;
       }
       
       location ~* \.php$ {
           fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
           include fastcgi.conf;
       }
   }
   ```

   ```sh
   sudo service nginx reload
   ```

配置结束。
