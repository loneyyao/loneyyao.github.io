---
title: 搬瓦工VPS搭建Hexo docker+nginx+ss+hexo
---
#### 1, 购买bwg vps 官网 https://bwh8.net/
#### 2, 配置vps ssh  root根目录新建.ssh/authorized_keys, 粘贴客户端的密码
#### 3, 安装docker  它只能用在 64 位的操作系统上 centeros需要注意内核版本
#### 4, docker部署ss https://hub.docker.com/u/teddysun/ go版本轻量速度快
#### 5, docker部署nginx 
       docker pull nginx
       docker run --name my_nginx -p 80:80  -d nginx
       先启动一个标准的nginx容器  然后把容器中的/etc/nginx/conf.d 和 nginx.conf文件复制到主机上 方便修改
       复制命令: docker cp a6:/etc/nginx/conf.d ~/nginx_config/conf/   //a6是容器地址  后面是本地路径,我建立在了本地根目录下
                docker cp a6:/etc/nginx/nginx.conf ~/nginx_config/conf/
       然后停止旧容器  删除旧容器  修改复制出来的配制文件为
       user  nginx;
       worker_processes  1;
       
       error_log  /var/log/nginx/error.log warn;
       pid        /var/run/nginx.pid;
       
       
       events {
           worker_connections  1024;
       }
       
       
       http {
            server {             //server为增加的部分
            listen       80;
            root   /home/git;
            server_name  localhost androidcc.com www.androidcc.com;
       
            location / {
            index  index.html index.htm;
              }
           }
       
       
           include       /etc/nginx/mime.types;
           default_type  application/octet-stream;
       
           log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                             '$status $body_bytes_sent "$http_referer" '
                             '"$http_user_agent" "$http_x_forwarded_for"';
       
           access_log  /var/log/nginx/access.log  main;
       
           sendfile        on;
           #tcp_nopush     on;
       
           keepalive_timeout  65;
       
           #gzip  on;
       
          # include /etc/nginx/conf.d/*.conf;      //可以注释掉 default.conf里面有nginx的默认server配置
       }

        重新启动一个新的nginx容器 并外挂其配置目录 和托管目录
        docker run --name my_nginx -p 80:80 -v /root/nginx_config/conf/nginx.conf:/etc/nginx/nginx.conf:ro -v /home/git/:/home/git -v /root/nginx_config/conf/conf.d/:/etc/nginx/conf.d -d nginx
        
   
#### 6, 安装git 创建git用户  
       adduser git
       git init --bare hexo.git   //建立git裸仓
       chown -R git:git hexo.git
       创建git用户ssh
       /home/git/.ssh/authorized_keys
       建立的裸仓中（即hexo.git文件夹中），找到hooks目录下的post-update.sample,重命名为post-update
       git --work-tree=/home/git --git-dir=/home/git/hexo.git checkout -f
     
       再搭建一个post-receive这个钩子，当git有收发的时候就会调用这个钩子。 在 ~/blog.git 裸库的 hooks文件夹中，
       新建post-receive文件。
       
       vim ~/home/git/hexo.git/hooks/post-receive
       git --work-tree=/path/to/www --git-dir=~/blog.git checkout -f
       保存后，要赋予这个文件可执行权限修改权限使其可执行
       chmod +x post-update
       
        修改/home/git文件夹的权限为711 赋予可执行权限 否则nginx报403 访问权限拒绝
        chmod 711 /home/git
  
#### 7, 本地hexo配置

        _config.yml下:
        deploy:
          type: git
          repo: 
            github: https://github.com/loneyyao/loneyyao.github.io.git
            bwg: ssh://git@X.X.X.X:端口号/home/git/hexo.git   //配置ssh访问
          branch: master 
          
#### 8, 本地写文章
        1, cd到博客目录下, vscode容易在工程根目录, 导致hexo命令无法正确执行
        hexo clean 
        hexo g -d
        
       
     
       
       
       