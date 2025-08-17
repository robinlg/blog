---
title: 经典且简单的企业级生产环境系统架构
date: 2021-07-31T08:29:17+00:00
categories:
  - 架构
tags:
  - 应用架构
---
<blockquote class="wp-block-quote">
  <p>
    各位读者好久不见，很长一段时间的周末都在忙着在为新工作做准备，所以没把时间用在写文章上。今天本也有安排， 可还是想抽点时间出来写一篇文章，对这一项目的技术栈做总结。
  </p>
</blockquote>

文章正式开始之前还是想简单说一句我找工作的想法。受到开源社区以及一些Youtube程序员UP主的影响，对于下一份工作，它应该是一份我自己喜欢而且有英语工作环境的工作。快乐的coding才能做出一个真正受欢迎的程序吧 😀 而且自己也不会感觉到很疲惫。期待有一天我也能拥有一个像Redis那样受欢迎的开源项目。

以下的内容是真实企业生产环境的项目搭建与配置，由于项目面向的人群比较特殊，因此系统的使用量不高，并发量也不大，所以没有采用比较复杂的设计。默认的配置也达到了压测的要求，所以没有对配置做过多的优化。


## <span class="ez-toc-section" id="%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84%E5%9B%BE"></span>系统架构图<span class="ez-toc-section-end"></span> {#1.wp-block-heading}

<img decoding="async" src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbcb0cde2f8244dc9b8a1cfa03abfe07~tplv-k3u1fbpfcp-watermark.awebp" alt="系統架構.png" /> 上图是系统架构图，是互联网公司里最常见的系统架构。这类系统往往处理并发量不高，而且业务也不复杂，所以做集群（包括应用系统集群，Redis集群，MySQL主从）的作用仅是保障它的高可用。系统里面有使用到消息队列，但要求不高，我们就直接用了Redis里的list结构来实现。简单的事情用简单的方式处理即可，对于简单的要求就没必要使用专业的MQ（RocketMQ, RabbitMQ），要不然反而增加系统的复杂性。

## <span class="ez-toc-section" id="%E6%80%BB%E4%BD%93%E8%AF%B7%E6%B1%82%E6%B5%81%E5%90%91%E5%9B%BE"></span>总体请求流向图<span class="ez-toc-section-end"></span> {#2.wp-block-heading}

<img decoding="async" src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a77e8f24dcf47c1bf490ef5334aa620~tplv-k3u1fbpfcp-watermark.awebp" alt="请求流向图.png" /> &nbsp;因为阿里云跟客户有合作协议，所以服务器资源都是来自于阿里云。通过上面的请求流向图，读者们也可以大致了解整个系统的网络架构。

## <span class="ez-toc-section" id="%E8%AF%A6%E7%BB%86%E8%AF%B7%E6%B1%82%E6%B5%81%E5%90%91%E5%9B%BE"></span>详细请求流向图<span class="ez-toc-section-end"></span> {#3.wp-block-heading}

<img decoding="async" src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4004ced04a9420198dc180ba1c2a4ad~tplv-k3u1fbpfcp-watermark.awebp" alt="详细请求流向图.png" /> 上图中的大矩形框表示一台服务器，其中有若干矩形框代表同一台服务器里装了若干组件。所有的请求都是经由Nginx发出，包括用户浏览HTML/图片/其他静态资源，JS请求服务端，所以上图的Nginx有三个出度。这里使用Redis哨兵模式而没有使用集群模式，主要是考虑到本系统对Redis的使用仅是作为Session共享和消息队列（消息的数量比较少），总体的数据量非常少，就使用简单一些的哨兵模式。而没有考虑使用Redis Cluster模式，将数据分散到不同服务器上（通过slot）。Redis Cluster模式的其他资料读者可自行查询其他资料。

读者通过上面的3张图对系统框架有了大概的了解后，下面附上所有组件的搭建步骤，项目名为iurc。

## <span class="ez-toc-section" id="%E7%BB%84%E4%BB%B6%E9%83%A8%E7%BD%B2%E6%90%AD%E5%BB%BA%E6%AD%A5%E9%AA%A4%E4%B8%8E%E9%85%8D%E7%BD%AE"></span>组件部署搭建步骤与配置<span class="ez-toc-section-end"></span> {#4.wp-block-heading}

### <span class="ez-toc-section" id="1_%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%9F%BA%E7%A1%80%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA"></span>1. 服务器基础环境搭建<span class="ez-toc-section-end"></span> {#4.1.wp-block-heading}

#### <span class="ez-toc-section" id="11_%E6%89%80%E6%9C%89%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%88%9B%E5%BB%BArobin%E7%94%A8%E6%88%B7"></span>1.1 所有服务器创建robin用户<span class="ez-toc-section-end"></span> {#4.1.1.wp-block-heading}

一共有9台服务器，假设IP为`10.210.111.12`&#8211;`10.210.111.20`。真实环境需要通过堡垒机登入，这里假设能直接登录服务器。

首先以root用户通过SSH工具登入服务器`10.210.111.12`，使用命令行创建`init_server.sh`脚本

首先以root用户通过SSH工具登入服务器`10.210.111.12`，使用命令行创建`init_server.sh`脚本

<pre class="wp-block-code"><code class="">vim init_server.sh

# 创建robin用户
adduser -m robin
# 修改用户密码
passwd robin
# 安装telent
yum -y install telnet</code></pre>

创建`send_all_servers.sh`脚本将`init_server.sh`脚本分发给所有服务器

<pre class="wp-block-code"><code class="">vim send_all_servers.sh

# 脚本中写入分发命令（此步骤的目的是节省创建用户的时间，但仍须登入每一台服务器执行该脚本）
scp $1 root@10.210.111.13:/root/
scp $1 root@10.210.111.14:/root/
scp $1 root@10.210.111.15:/root/
scp $1 root@10.210.111.16:/root/
scp $1 root@10.210.111.17:/root/
scp $1 root@10.210.111.18:/root/
scp $1 root@10.210.111.19:/root/
scp $1 root@10.210.111.20:/root/</code></pre>

执行`send_all_servers.sh`脚本（暂未做免密登录，执行脚本后需要输入目标服务器密码）

<pre class="wp-block-code"><code class="">chmod +x ./send_all_servers.sh
bash ./send_all_servers.sh ./init_server.sh</code></pre>

在所有服务器执行`init_server.sh`脚本

<pre class="wp-block-code"><code class="">chmod +x ./init_server.sh
bash ./init_server.sh</code></pre>

#### <span class="ez-toc-section" id="12_%E6%89%80%E6%9C%89%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%8E%88%E6%9D%83robin%E7%94%A8%E6%88%B7"></span>1.2 所有服务器授权robin用户<span class="ez-toc-section-end"></span> {#4.1.2.wp-block-heading}

添加`sudoers`文件可寫權限

<pre class="wp-block-code"><code class="">chmod u+w /etc/sudoers</code></pre>

修改`sudoers`文件

<pre class="wp-block-code"><code class="">vim /etc/sudoers</code></pre>

在sudoers文件中找到如下图<figure class="wp-block-image">

<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/913e4118b58342079cad8aeed3e66bac~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> </figure> 

在root行下方添加如下內容

<pre class="wp-block-code"><code class="">robin ALL=(ALL) ALL</code></pre>

收回`sudoers`文件可写权限

<pre class="wp-block-code"><code class="">chmod u-w /etc/sudoers</code></pre>

⚠️ 所有ECS都需要登入执行授权操作

#### <span class="ez-toc-section" id="13_%E4%BF%AE%E5%A4%8D%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%BB%E6%9C%BA%E5%90%8D"></span>1.3 修复服务器主机名<span class="ez-toc-section-end"></span> {#4.1.3.wp-block-heading}

修改`10.210.111.12`服务器主机名

<pre class="wp-block-code"><code class="">hostnamectl set-hostname nginx_01
reboot</code></pre>

建议以此修改方式，将所有主机名改为此服务器主要提供的服务，在后续方便运维。我设置的主机名如下：<figure class="wp-block-table">

| IP地址          | 系統        | 主机名      |
| ------------- | --------- | -------- |
| 10.210.111.12 | CentOS7.x | nginx_01 |
| 10.210.111.13 | CentOS7.x | nginx_02 |
| 10.210.111.14 | CentOS7.x | redis_01 |
| 10.210.111.15 | CentOS7.x | redis_02 |
| 10.210.111.16 | CentOS7.x | redis_03 |
| 10.210.111.17 | CentOS7.x | db_01    |
| 10.210.111.18 | CentOS7.x | db_02    |
| 10.210.111.19 | CentOS7.x | app_02   |
| 10.210.111.20 | CentOS7.x | app_02   |</figure> 

### <span class="ez-toc-section" id="2_Nginx%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE"></span>2. Nginx安装及配置<span class="ez-toc-section-end"></span> {#4.2.wp-block-heading}

服务器配置<figure class="wp-block-table">

| IP地址          | 系统        | 功能    |
| ------------- | --------- | ----- |
| 10.210.111.12 | CentOS7.x | Nginx |
| 10.210.111.13 | CentOS7.x | Nginx |</figure> 

#### <span class="ez-toc-section" id="21_%E5%AE%89%E8%A3%85Nginx"></span>2.1 安装Nginx<span class="ez-toc-section-end"></span> {#4.2.1.wp-block-heading}

使用**SSH工具**以robin用戶登入服務器**nginx_01**`10.210.111.12`，下載Nginx並解壓。

<pre class="wp-block-code"><code class="">cd /home/robin
wget http://nginx.org/download/nginx-1.20.1.tar.gz
tar -zxvf nginx-1.20.1.tar.gz && cd nginx-1.20.1</code></pre>

安装预编译环境

<pre class="wp-block-code"><code class="">sudo yum update
sudo yum -y install gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel</code></pre>

编译安装用以下配置替換`/usr/local/nginx/conf/nginx.conf`裡的內容

<pre class="wp-block-code"><code class="">cd /home/robin/nginx-1.20.1
./configure --prefix=/usr/local/nginx
make
sudo make install</code></pre>

#### <span class="ez-toc-section" id="22_%E9%85%8D%E7%BD%AENginx"></span>2.2 配置Nginx<span class="ez-toc-section-end"></span> {#4.2.2.wp-block-heading}

用以下配置替换`/usr/local/nginx/conf/nginx.conf`里的内容

<pre class="wp-block-code"><code class="">worker_processes  2;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;
        
	location @router{
            rewrite ^.*$ /iurc/index.html break;
        }

        location ^~/iurc {
            root   /usr/local/nginx/html;
            try_files $uri $uri/ @router;
        }

	location ^~/iurc/api {
            proxy_pass http://iurcProxy/iurc/api/channel/http.do;
            proxy_send_timeout 1800;
            proxy_read_timeout 1800;
            proxy_connect_timeout 1800;
            client_max_body_size 2048m;
            proxy_set_header  Host              $http_host;
            proxy_set_header  X-Real-IP   $remote_addr;
            proxy_set_header  X-Real-Port       $remote_port;
            proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
        }

	location @router_admin{
            rewrite ^.*$ /iurc-admin/index.html break;
        }

        location ^~/iurc-admin {
            allow 192.168.10.0/24;
            allow 192.168.12.0/24;
            allow 192.168.30.0/24;
            allow 192.168.160.0/24;
            deny all;
            root   /usr/local/nginx/html;
            try_files $uri $uri/ @router_admin;
        }
	
        location ^~/iurc-admin/api {
            allow 192.168.10.0/24;
            allow 192.168.12.0/24;
            allow 192.168.30.0/24;
            allow 192.168.160.0/24;
            deny all;
            proxy_pass http://iurcAdminProxy/iurc/admin/api/channel/http.do;
            proxy_send_timeout 1800;
            proxy_read_timeout 1800;
            proxy_connect_timeout 1800;
            client_max_body_size 2048m;
            proxy_set_header  Host              $http_host;
            proxy_set_header  X-Real-IP   $remote_addr;
            proxy_set_header  X-Real-Port       $remote_port;
            proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
        }

        location = /50x.html {
            root   html;
        }

    }

    upstream iurcProxy {
        server 10.210.111.19:30068;
        server 10.210.111.20:30068;
    }
    
    upstream iurcAdminProxy {
        server 10.210.111.19:30070;
        server 10.210.111.20:30070;
    }
}</code></pre>

#### <span class="ez-toc-section" id="23_%E5%90%AF%E5%8A%A8Nginx"></span>2.3 启动Nginx<span class="ez-toc-section-end"></span> {#4.2.3.wp-block-heading}

<pre class="wp-block-code"><code class="">sudo /usr/local/nginx/sbin/nginx</code></pre>

⚠️ 在**nginx_02**`10.210.111.13`服务器上按照步骤2.1 &#8211; 2.3安装配置Nginx

#### <span class="ez-toc-section" id="24_%E5%88%9B%E5%BB%BAHTML%E5%AD%98%E6%94%BE%E7%9B%AE%E5%BD%95"></span>2.4 创建HTML存放目录<span class="ez-toc-section-end"></span> {#4.2.4.wp-block-heading}

<pre class="wp-block-code"><code class="">sudo mkdir /usr/local/nginx/html/iurc
sudo mkdir /usr/local/nginx/html/iurc-admin</code></pre>

若修改了`/usr/local/nginx/conf/nginx.conf`里的内容，可用如下命令重新启动Nginx，使新配置生效。

<pre class="wp-block-code"><code class="">sudo /usr/local/nginx/sbin/nginx -s reload</code></pre>

### <span class="ez-toc-section" id="3_Redis%E9%9B%86%E7%BE%A4%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE%EF%BC%88%E5%93%A8%E5%85%B5%E6%A8%A1%E5%BC%8F%EF%BC%89"></span>3. Redis集群安装及配置（哨兵模式）<span class="ez-toc-section-end"></span> {#4.3.wp-block-heading}

服务器配置<figure class="wp-block-table">

| IP地址          | 系统        | 功能        |
| ------------- | --------- | --------- |
| 10.210.111.14 | CentOS7.x | Redis, 哨兵 |
| 10.210.111.15 | CentOS7.x | Redis, 哨兵 |
| 10.210.111.16 | CentOS7.x | Redis, 哨兵 |</figure> 

#### <span class="ez-toc-section" id="31_%E5%AE%89%E8%A3%85Redis"></span>3.1 安装Redis<span class="ez-toc-section-end"></span> {#4.3.1.wp-block-heading}

以robin用户登入服务器**redis_01**`10.210.111.14`，下载Redis并解压。

<pre class="wp-block-code"><code class="">cd /home/robin
wget https://download.redis.io/releases/redis-5.0.9.tar.gz
tar -zxvf redis-5.0.9.tar.gz && cd redis-5.0.9</code></pre>

安装预编译环境及安装

<pre class="wp-block-code"><code class="">sudo yum update
sudo yum -y install gcc
make
sudo make install PREFIX=/usr/local/redis</code></pre>

⚠️ 在**redis_02**`10.210.111.15`服务器上按照步骤3.1安装配置Redis

⚠️ 在**redis_03**`10.210.111.16`服务器上按照步骤3.1安装配置Redis

#### <span class="ez-toc-section" id="32_%E9%85%8D%E7%BD%AERedis"></span>3.2 配置Redis<span class="ez-toc-section-end"></span> {#4.3.2.wp-block-heading}

在`10.210.111.14`中，从redis源码目录中复制redis.conf至redis的安装目录

<pre class="wp-block-code"><code class="">sudo cp /home/robin/redis-5.0.9/redis.conf /usr/local/redis/bin/ && cd /usr/local/redis/bin/</code></pre>

在`10.210.111.14`中，修改redis.conf文件：

<pre class="wp-block-code"><code class="">sudo vim redis.conf</code></pre>

将`daemonize no`改为`daemonize yes`<figure class="wp-block-image">

<img decoding="async" src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/448b5af582314f7c85a03853f7a8dfb8~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> </figure> 

把默认的回环地址注释掉，默认不能被其他机器访问<figure class="wp-block-image">

<img decoding="async" src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79354b7c261b44d289622afa382710a6~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> </figure> 

关闭保护模式，由`yes`改为`no`<figure class="wp-block-image">

<img decoding="async" src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ccd1537f9af7417082cf888466414e2e~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> </figure> 

将在`10.210.111.14`中修改完成的redis.conf文件分发到`10.210.111.15`及`10.210.111.16`中

<pre class="wp-block-code"><code class="">scp /usr/local/redis/bin/redis.conf root@10.210.111.15:/usr/local/redis/bin/
# 此处会要求输入10.210.111.15的root密码

scp /usr/local/redis/bin/redis.conf root@10.210.111.16:/usr/local/redis/bin/
# 此处会要求输入10.210.111.16的root密码</code></pre>

在`10.210.111.15`中，编辑redis.conf

<pre class="wp-block-code"><code class="">sudo vim /usr/local/redis/bin/redis.conf
#在最后一行加入如下内容
replicaof 10.210.111.14 6379</code></pre>

在`10.210.111.16`中，编辑redis.conf

<pre class="wp-block-code"><code class="">sudo vim /usr/local/redis/bin/redis.conf
#在最后一行加入如下内容
replicaof 10.210.111.14 6379</code></pre>

#### <span class="ez-toc-section" id="33_%E9%85%8D%E7%BD%AERedis_Sentinel"></span>3.3 配置Redis Sentinel<span class="ez-toc-section-end"></span> {#4.3.3.wp-block-heading}

在`10.210.111.14`中，安装redis-sentinel

<pre class="wp-block-code"><code class="">sudo mkdir -p /usr/local/redis-sentinel
sudo cp -r /usr/local/redis/* /usr/local/redis-sentinel</code></pre>

在`10.210.111.14`中，复制sentinel.conf

<pre class="wp-block-code"><code class="">sudo cp /home/robin/redis-5.0.9/sentinel.conf /usr/local/redis-sentinel/bin</code></pre>

在`10.210.111.14`中，修改sentinel.conf

<pre class="wp-block-code"><code class="">sudo vim /usr/local/redis-sentinel/sentinel.conf

#将daemonize由no改为yes
daemonize yes
#指定多少毫秒后，主节点没有应答sentinel。此时，sentinel主观上认为主节点下线，默认30000毫秒，改为3000毫秒
sentinel down-after-milliseconds mymaster 3000</code></pre>

将`10.210.111.14`的sentinel复制到`10.210.111.15`和`10.210.111.16`中

<pre class="wp-block-code"><code class="">scp -r /usr/local/redis-sentinel/ root@10.210.111.15:/usr/local
scp -r /usr/local/redis-sentinel/ root@10.210.111.16:/usr/local</code></pre>

在`10.210.111.15`中，修改redis.conf文件

<pre class="wp-block-code"><code class="">vim /usr/local/redis-sentinel/bin/sentinel.conf

# 将sentinel监控的redis主节点ip改为10.210.111.14
sentinel monitor mymaster 10.210.111.14 6379 2</code></pre>

在`10.210.111.16`中，修改redis.conf文件

<pre class="wp-block-code"><code class="">sudo vim /usr/local/redis-sentinel/bin/sentinel.conf

# 将sentinel监控的redis主节点ip改为10.210.111.14
sentinel monitor mymaster 10.210.111.14 6379 2</code></pre>

#### <span class="ez-toc-section" id="34_%E8%AE%BE%E7%BD%AE%E5%BC%80%E6%9C%BA%E5%90%AF%E5%8A%A8Redis%E5%8F%8ARedis_Sentinel"></span>3.4 设置开机启动Redis及Redis Sentinel<span class="ez-toc-section-end"></span> {#4.3.4.wp-block-heading}

在`10.210.111.14`中，**添加Redis开机启动服务**

<pre class="wp-block-code"><code class="">sudo vim /etc/systemd/system/redis.service

# 复制粘贴以下内容
[Unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/redis/bin/redis-server /usr/local/redis/bin/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target</code></pre>

在`10.210.111.14`中，**添加Redis Sentinel开机启动服务**

<pre class="wp-block-code"><code class="">sudo vim /etc/systemd/system/redis-sentinel.service

[Unit]
Description=redis-sentinel
After=network.target

# 复制粘贴以下内容
[Service]
Type=forking
ExecStart=/usr/local/redis-sentinel/bin/redis-sentinel /usr/local/redis-sentinel/bin/sentinel.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target</code></pre>

配置解释如下

<pre class="wp-block-code"><code class=""># [Unit]: 服务单元
# Description: 描述服务
# After: 描述服务的类别

# [Serivce]: 服务运行参数的设置
# Type=forking: 是后台运行的形式
# ExecStart: 服务的具体运行命令
# ExecReload: 服务的重启命令
# ExecStop: 服务的停止命令
# PrivateTmp=True: 表示给服务分配独立的临时空间
# ⚠️ 注意：[Service]的启动、重启、停止命令全部要求使用绝对路径

# [Install]: 运行级别下服务安装的相关设置，可设置为多用户，即系统运行级别为3</code></pre>

⚠️ 在`10.210.111.15`和`10.210.111.16`中重复以上表示**添加Redis开机启动服务**和**添加Redis Sentinel开机启动服务**步骤。

#### <span class="ez-toc-section" id="35_%E5%90%AF%E5%8A%A8Redis%E5%8F%8ARedis_Sentinel"></span>3.5 启动Redis及Redis Sentinel<span class="ez-toc-section-end"></span> {#4.3.5.wp-block-heading}

在`10.210.111.14`中，**启动redis并加入开机自启动**

<pre class="wp-block-code"><code class="">sudo systemctl daemon-reload
sudo systemctl start redis.service 
sudo systemctl enable redis.service</code></pre>

在`10.210.111.14`中，**启动Redis sentinel并加入开机自启动**

<pre class="wp-block-code"><code class="">sudo systemctl start redis-sentinel.service 
sudo systemctl enable redis-sentinel.service</code></pre>

⚠️ 在`10.210.111.15`和`10.210.111.16`中重复以上表示**启动redis并加入开机自启动**和**启动Redis sentinel并加入开机自启动**步骤。

#### <span class="ez-toc-section" id="36_%E6%B5%8B%E8%AF%95"></span>3.6 测试<span class="ez-toc-section-end"></span> {#4.3.6.wp-block-heading}

在主Redis`10.210.111.14`进入redis控制台

<pre class="wp-block-code"><code class="">sudo /usr/local/redis/bin/redis-cli</code></pre>

输入命令

<pre class="wp-block-code"><code class="">info</code></pre>

若出现下图结果，连接了2个slaves则表示Redis主从（哨兵模式）安装成功

<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2a69a459e074a629cd0d2302113e6bd~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> ⚠️ 输入info命令出来的结果由于网络或者同步状态的情况，`connected_slaves`不一定实时为2，可以多查看几次。

### <span class="ez-toc-section" id="4_MySQL%E5%AE%89%E8%A3%85%E5%8F%8A%E9%85%8D%E7%BD%AE%EF%BC%88%E4%B8%BB%E4%BB%8E%E6%A8%A1%E5%BC%8F%EF%BC%89"></span>4. MySQL安装及配置（主从模式）<span class="ez-toc-section-end"></span> {#4.4.wp-block-heading}

服务器配置<figure class="wp-block-table">

| IP地址          | 系统        | 功能   |
| ------------- | --------- | ---- |
| 10.210.111.17 | CentOS7.x | 主數據庫 |
| 10.210.111.18 | CentOS7.x | 從數據庫 |</figure> 

#### <span class="ez-toc-section" id="41_%E5%AE%89%E8%A3%85MySQL"></span>4.1 安装MySQL<span class="ez-toc-section-end"></span> {#4.4.1.wp-block-heading}

以robin用户登入服务器**db_01**`10.210.111.17`，通过YUM源方式安装MySQL。

下面链接很详细的官网安装步骤，读者照里面这个步骤安装MySQL即可

<blockquote class="wp-block-quote">
  <p>
    <a href="https://link.juejin.cn?target=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Flinux-installation-yum-repo.html" target="_blank" rel="noreferrer noopener">dev.mysql.com/doc/refman/…</a>
  </p>
</blockquote>

#### <span class="ez-toc-section" id="42_%E8%AE%BE%E7%BD%AEMySQL%E8%BF%9C%E7%A8%8B%E8%BF%9E%E6%8E%A5"></span>4.2 设置MySQL远程连接<span class="ez-toc-section-end"></span> {#4.4.2.wp-block-heading}

设置MySQL只能两个APP SERVER`10.210.111.19`和`10.210.111.20`连接

<pre class="wp-block-code"><code class=""># 在MYSQL命令行中输入以下命令
GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.210.111.19' IDENTIFIED BY 'P@ssw0rd' WITH GRANT OPTION; 
GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.210.111.20' IDENTIFIED BY 'P@ssw0rd' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.210.111.14' IDENTIFIED BY 'P@ssw0rd' WITH GRANT OPTION; 
GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.210.111.15' IDENTIFIED BY 'P@ssw0rd' WITH GRANT OPTION;
FLUSH PRIVILEGES; </code></pre>

⚠️ 至此已完成`10.210.111.17`的MySQL安装，请登录`10.210.111.18`参考**步骤[4.1 &#8211; 4.2]** 在`10.210.111.18`上安装MySQL。

#### <span class="ez-toc-section" id="43_%E8%AE%BE%E7%BD%AE%E4%B8%BB%E4%BB%8E%E5%90%8C%E6%AD%A5"></span>4.3 设置主从同步<span class="ez-toc-section-end"></span> {#4.4.3.wp-block-heading}

⚠️ 将`10.210.111.17`设置为主数据库，将`10.210.111.18`设置为从数据库

<pre class="wp-block-code"><code class="">mysql -uroot -p
# 輸入密碼
CREATE USER 'slave'@'%' IDENTIFIED BY 'slaveP@ssw0rd';
GRANT ALL PRIVILEGES ON *.* TO 'slave'@"%" IDENTIFIED BY "slaveP@ssw0rd";</code></pre>

修改`10.210.111.17`的my.cnf配置文件

<pre class="wp-block-code"><code class="">sudo vim /etc/my.cnf
# 加入两行内容
log-bin=mysql-bin
server-id=1</code></pre><figure class="wp-block-image">

<img decoding="async" src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e64c6106dbdd428397660a636320b067~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> </figure> 

说明

<pre class="wp-block-code"><code class=""># 开启二进制日志，该日志是在事务提交时写日志文件的。默认大小是1G，后面加001,002这样的后缀顺加。
log-bin=mysql-bin

# 唯一标识主机，mysql主从每个mysql实例配置都不一样就行。这个值默认是0，如果是0，主服务器拒绝任何从服务器的连接。
server-id=1

# 其他配置（不是必须配置的）：

# 指定mysql的binlog日志记录哪个db，配置需要同步的数据库，可以配置多个，如果没有此配置项则同步全部。
binlog-do-db=db_001（主数据库配置）

# 配置不同步的数据库，可以配置多个。
binlog-ignore-db=mysql（主数据库配置）

# 配置binlog的格式
binlog_format=mixed

# 配置是否只读  0代表不只读，1代表只读
read-only=0

# 用于设定双主情况下自增列的ID冲突使用的，主要用来设置自增步长
auto-increament-increment=10

# 表示这台服务器的序号，从1开始，不超过auto-increament-increment
auto-increment-offset=1</code></pre>

重启`10.210.111.17`的MySQL

<pre class="wp-block-code"><code class="">sudo systemctl restart mysqld</code></pre>

修改`10.210.111.18`的my.cnf配置文件

<pre class="wp-block-code"><code class="">sudo vim /etc/my.cnf
# 加入两行内容
server-id=2
relay_log=mysql-relay-bin</code></pre>

重启`10.210.111.18`的MySQL

<pre class="wp-block-code"><code class="">systemctl restart mysqld.service</code></pre>

获取 `10.210.111.17`MySQL的binlog信息

<pre class="wp-block-code"><code class="">mysql -uroot -p
# 输入密码
show master status;</code></pre><figure class="wp-block-image">

<img decoding="async" src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06b99bff95224cb7a51afa18ac1bdad3~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> </figure> 

进入`10.210.111.18`的MySQL设置与主库同步

<pre class="wp-block-code"><code class="">mysql -uroot -p
# 输入密码
change master to master_host='10.210.111.17',master_user='slave',master_password='slaveP@ssw0rd',master_log_file='mysql-bin.000001', master_log_pos=154;</code></pre>

⚠️ 注意以下说明

<pre class="wp-block-code"><code class="">master_user和master_password是上面步骤中在主服务器上创建的用户名“slave”及其密码

master_log_file是在主服务器上查询出来的“MySQL binlog信息”中的File字段值。如上图中的mysql-bin.000001

master_log_pos是在主服务器上查询出来的“MySQL binlog信息”中的Position字段值。如上图中的154</code></pre>

开启`10.210.111.18`的从MySQL slave线程

<pre class="wp-block-code"><code class="">start slave;</code></pre>

查看`10.210.111.18`的从MySQL slave状态

<pre class="wp-block-code"><code class="">show slave status \G</code></pre>

<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ccea649225364970a7368b5153ae7ed2~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 看到红框中两个Yes，就代表从从服务器已经成功从主服务器同步数据。

#### <span class="ez-toc-section" id="44_%E4%B8%BB%E4%BB%8E%E9%AA%8C%E8%AF%81"></span>4.4 主从验证<span class="ez-toc-section-end"></span> {#4.4.4.wp-block-heading}

在`10.210.111.17`主MySQL執行建庫建表操作，驗證主從模式部署成功

<pre class="wp-block-code"><code class="">mysql -uroot -p
# 输入密码
create database test;
use test;
create table user_test( id int comment'ID',name VARCHAR(20) comment'名称', create_time timestamp DEFAULT now() comment'创建时间' );
insert INTO user_test value(1,"A",NOW());</code></pre>

在`10.210.111.18`从MySQL查看是否有在主MySQL创建的表和数据

<pre class="wp-block-code"><code class="">mysql -uroot -p
# 输入密码
show databases;
use test;
show tables;
select * from user_test;</code></pre>

从结果可以看出，数据已经从主MySQL同步至从MySQL，证明主从模式已经部署成功。接下来在`10.210.111.17`主MySQL删除测试数据即可。

<pre class="wp-block-code"><code class="">drop database test;</code></pre>

### <span class="ez-toc-section" id="5%E9%9D%99%E6%80%81%E8%B5%84%E6%BA%90%E5%90%8C%E6%AD%A5%E5%AE%89%E8%A3%85"></span>5.静态资源同步安装<span class="ez-toc-section-end"></span> {#4.5.wp-block-heading}

这部分的内容仅供参考，因为现在看回这个设计其实是有些问题的。当初想通过Nginx做动静分离，静态文件直接通过Nginx访问，文件备份通过“静态资源同步服务”。对于公开的静态资源当然没问题，但是如果用户上传的静态资源是不能公开的，这里就存在了很大的风险，攻击者很可能通过Nginx直接下载用户的隐私资料。

综上所述得出两条建议：

  * 对于隐私文件，有条件的话可以使用OSS服务。如果无条件可参考如下的静态资源同步服务，我认为除了我下面的服务外，应该还会有更好的选择，读者可以自行查找；
  * 对于可公开的静态资源可以采用动静分离方案，通过Nginx直接获取。对于隐私文件，切记不可通过配置Nginx直接。

服务器配置<figure class="wp-block-table">

| IP地址          | 系统        | 功能       |
| ------------- | --------- | -------- |
| 10.210.111.13 | CentOS7.x | rsync服務端 |
| 10.210.111.12 | CentOS7.x | rsync客戶端 |</figure> 

#### <span class="ez-toc-section" id="51_rsync%E6%9C%8D%E5%8B%99%E7%AB%AF"></span>5.1 rsync服務端<span class="ez-toc-section-end"></span> {#4.5.1.wp-block-heading}

在`10.210.111.13`安裝rsync

<pre class="wp-block-code"><code class="">cd /opt
sudo yum install -y rsync</code></pre>

配置/etc/rsyncd.conf

<pre class="wp-block-code"><code class=""># 纪录传输文件日志
transfer logging = yes
# 指定日志文件
log file = /var/log/rsyncd.log
# pid文件路径
pid file = /var/run/rsyncd.pid
# 锁路径
lock file = /var/run/rsync.lock
# 运行rsync守护进程的用户
uid = robin
# 运行rsync守护进程的组
gid = robin
# 是否允许chroot，用于提升安全性。客户端连接模块，首先chroot到模块path参数指定的目录下，chroot为yes是必须使用root权限，且不能备份path路径外的连接文件
use chroot = yes
# 忽略错误
ignore errors
# 只读
read only = no
# 设置超时时间，单位秒
timeout = 60
ignore nonreadable = yes

# 模块，可配置多个
[sync_file]
# 模块的根目录，同步目录，需要注意权限
path = /home/robin/static
# 模块认证的用户名称，可使用空格或逗号隔开多个用户名
auth users = sync
# 模块验证密码文件，可放在全局配置里
secrets file = /etc/rsyncd.secrets
# 设定白名单，可指定IP段（172.18.50.1/255.255.255.0），各IP段用空格分开
hosts allow = 10.210.111.12
hosts deny = *
# 是否允许列出模块内容
list = no</code></pre>

新建/etc/rsyncd.secrets，此处用robin用户作为使用同步服务的用户

<pre class="wp-block-code"><code class=""># 需要使用root角色执行echo "robin:infoP@ssw0rd" &gt; /etc/rsyncd.secrets
sudo -i
#输入密码使用root
echo "sync:123456" &gt; /etc/rsyncd.secrets
#再转回使用普通用户
su robin</code></pre>

设置文件权限

<pre class="wp-block-code"><code class="">sudo chmod 600 /etc/rsyncd.secrets</code></pre>

启动服务

<pre class="wp-block-code"><code class="">sudo systemctl restart rsyncd</code></pre>

#### <span class="ez-toc-section" id="52_rsync%E5%AE%A2%E6%88%B6%E7%AB%AF"></span>5.2 rsync客戶端<span class="ez-toc-section-end"></span> {#4.5.2.wp-block-heading}

在`10.210.111.12`安裝rsync

<pre class="wp-block-code"><code class="">cd /opt
sudo yum install -y rsync</code></pre>

新建/etc/rsync.passwd，內容如下：

<pre class="wp-block-code"><code class=""># 需要使用root角色执行echo "infoP@ssw0rd" &gt; /etc/rsync.passwd
sudo -i
#输入密码使用root
echo "123456" &gt; /etc/rsync.passwd
#再转回使用普通用户
su robin</code></pre>

⚠️ 客户端rsync只需要密码，此处密码填写robin用户的密码

更改权限

<pre class="wp-block-code"><code class="">sudo chmod 600 /etc/rsync.passwd</code></pre>

安裝inotify

<blockquote class="wp-block-quote">
  <p>
    inotify-tools工具检测文件增加、删除、修改。
  </p>
</blockquote>

<pre class="wp-block-code"><code class="">sudo yum install -y inotify-tools</code></pre>

编写启动脚本inotify_start.sh

<pre class="wp-block-code"><code class="">cd /home/robin</code></pre>

脚本内容如下：

<pre class="wp-block-code"><code class="">#!/bin/bash

Path=/home/robin/static
# 此處host填rsync服務器的IP
Server=10.210.111.13
User=sync
module=sync_file

/usr/bin/inotifywait -mrq --format '%w%f' -e create,close_write,delete $Path  | while read line  
do
    if [ -f $line ];then
        rsync -az $line --delete ${User}@${Server}::${module} --password-file=/etc/rsync.passwd
    else
        cd $Path &&\
        rsync -az ./ --delete ${User}@${Server}::${module} --password-file=/etc/rsync.passwd
    fi
done</code></pre>

测试命令

<pre class="wp-block-code"><code class="">sudo rsync -avz --delete /home/robin/static/ sync@10.210.111.13::sync_file --password-file=/etc/rsync.passwd</code></pre>

#### <span class="ez-toc-section" id="53_%E7%BC%96%E5%86%99%E6%9C%8D%E5%8A%A1%E8%87%AA%E5%90%AF%E5%8A%A8%E8%84%9A%E6%9C%AC"></span>5.3 编写服务自启动脚本<span class="ez-toc-section-end"></span> {#4.5.3.wp-block-heading}

编写inotifyd.service并将其拷贝到/usr/lib/systemd/system/目录下，内容如下

<pre class="wp-block-code"><code class="">[Unit]
Description=inotify rsync script
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=root
Group=root
PIDFile=/var/run/inotify.pid
ExecStart=/usr/bin/nohup /home/robin/inotify_start.sh &gt; /home/robin/inotify.log 2&gt;&1 &
ExecReload=
ExecStop=/usr/bin/ps -ef | /usr/bin/grep inotify | /usr/bin/grep -v grep | /usr/bin/awk '{print $2}' | /usr/bin/xargs /usr/bin/kill -9
PrivateTmp=true

[Install]
WantedBy=multi-user.target</code></pre>

启动服务

<pre class="wp-block-code"><code class="">sudo systemctl start inotifyd.service</code></pre>

允许自动启动

<pre class="wp-block-code"><code class="">sudo systemctl enable inotifyd.service</code></pre>

重新读取重启脚本

<pre class="wp-block-code"><code class="">sudo systemctl daemon-reload</code></pre>

PS：以上是我在掘金写的文章，所以图片带水印。