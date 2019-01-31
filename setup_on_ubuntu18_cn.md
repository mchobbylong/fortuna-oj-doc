# Fortuna OJ 部署指南

以下指南将在纯净的 Ubuntu 18.04.1 LTS 操作系统上部署 fortuna-oj。

PS：Fortuna OJ 现已[开源](https://github.com/roastduck/fortuna-oj)，部署请暂时使用 [new-env](https://github.com/roastduck/fortuna-oj/tree/new-env) 分支



## 配置环境

1. 更换 apt 源为清华 Tuna（可选）

   ```sh
   # TDB
   ```

2. 安装相关依赖

   ```sh
   sudo apt-get install -y git python
   ```

3. 安装 nginx

   ```sh
   sudo apt-get install -y nginx
   ```

4. 安装 MariaDB

   ```sh
   sudo apt-get install -y mariadb-server
   sudo service mariadb restart
   ```

5. 安装 redis

   ```sh
   sudo apt-get install -y redis
   ```

6. 安装 php 7.2 以及部分 php 插件

   ```sh
   sudo apt-get install -y software-properties-common apt-transport-https lsb-release ca-certificates
   sudo add-apt-repository ppa:ondrej/php
   sudo apt-get update
   sudo apt-get install -y php7.2-fpm php7.2-mysql php7.2-curl php7.2-gd php7.2-mbstring php7.2-xml php7.2-xmlrpc php7.2-zip php7.2-opcache php7.2-redis
   sudo service php7.2-fpm restart
   ```

7. 配置 redis 监听 Unix Socket

   ```sh
   # TDB
   # 应修改 /etc/redis/redis.conf，找到以下两行，去除注释符号并作修改；或直接添加
   # 
   # unixsocket /var/run/redis/redis-server.sock
   # unixsocketperm 770
   sudo usermod -aG redis www-data
   sudo service redis-server restart
   ```

8. 重启操作系统以应用变更（可选）



## 部署 fortuna-oj

1. 从 GitHub 拉取 fortuna-oj

   ```sh
   cd /var/www
   sudo mkdir foj
   sudo chown www-data:www-data foj
   sudo -u www-data git clone https://github.com/mchobbylong/fortuna-oj foj
   cd foj
   sudo -u www-data git checkout new-env
   ```

2. 导入相关数据库

   ```sh
   # TBD
   # 在导入的 sql 文件中默认新建一个数据库用户：
   # 用户名 foj
   # 密码 foj
   # 以及一个 fortuna-oj 专用数据库 foj
   # 上述三项应在 sh 脚本中成为询问项
   sudo mysql < /var/www/foj/migrate/full.sql
   ```

3. 根据第 2 步的数据库相关信息，修改 fortuna-oj 配置文件

   ```sh
   cd /var/www/foj/overriding_config
   sudo -u www-data cp local.php.example local.php
   sudo -u www-data ./gen_secret.py
   ```

   ```php
   # /var/www/foj/overriding_config/local.php
   # cookie_path 与 nginx 配置中的 root 有关，注意修改
   
   $assign_to_config['cookie_path'] = '/';
   $db['default']['database'] = 'foj';
   ```

4. 配置 nginx 网站配置

   ```nginx
   # /etc/nginx/sites-enabled/foj.conf
   
   server {
       listen 80;
       listen [::]:80;
       
       root /var/www/foj;
       index index.html index.php;
       
       location / {
           try_files $uri $uri/ /index.php;
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



## 已知问题

### Bug

- 【重要】创建第一个用户后无法登录，提示密码验证不通过