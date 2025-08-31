---
title: Nginx做正向代理
date: 2021-02-04T15:24:03+00:00
categories:
  - Nginx
---
近日，朋友在公司摸鱼时发现笔记本在使用公司的WI-FI后无法进入B站。于是乎在群里“拍了拍”我。本着科技服务生活的精神，我欣然地接受了这个请求 😀

<!--more-->

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2021/02/image.png" alt="" class="wp-image-769" width="414" height="165" /><figcaption>“请求中&#8230;“</figcaption></figure>
</div>

使用Nginx在我的云服务器（国内节点）上搭了个正向代理服务器，笔记本的浏览器只需将代理服务器IP和代理端口设置为云服务器外网IP和Nginx里配置的监听端口即可。详细过程如下：

登陆云服务器（我的是CentOS 7.x），下载Nginx的tar包：

<pre class="wp-block-preformatted"># 进入登陆账号的home目录下，我的是/home/robin<br />cd ~<br /># 下载Nginx tar包<br />wget http://nginx.org/download/nginx-1.18.0.tar.gz<br /># 解压tar包<br />tar -zxvf nginx-1.18.0.tar.gz && cd nginx-1.18.0</pre>

安装预编译环境：

<pre class="wp-block-preformatted">#升级所有包同时也升级软件和系统内核<br />sudo yum update<br /># 安装依赖环境<br />sudo yum -y install gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel</pre>

<p class="has-vivid-cyan-blue-color has-text-color">
  Nginx作为反向代理服务器，官方一直没有支持HTTP CONNECT方法。但是基于Nginx的模块化、可扩展性好的特性，阿里的@chobits提供了<a href="https://github.com/chobits/ngx_http_proxy_connect_module">ngx_http_proxy_connect_module</a>模块，来支持HTTP CONNECT方法，从而让Nginx可以扩展为正向代理。使用此模块需要pcre环境，如果没安装pcre，在后面的make命令执行后，会有缺少pcre的报错信息。
</p>

将[ngx\_http\_proxy\_connect\_module][1]克隆至服务器上：

<pre class="wp-block-preformatted"># 进入登陆账号的home目录下，我的是/home/robin<br />cd ~<br /># 克隆项目至本地<br />git clone https://github.com/chobits/ngx_http_proxy_connect_module.git </pre>

模块克隆好后，需要根据Nginx的版本给最初的文件（最初的文件是：/home/robin/ngx\_http\_proxy\_connect\_module/ngx\_http\_proxy\_connect\_module.c）打补丁，我的Nginx是1.19.0，所以打的补丁版本如下红框：

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2021/02/image-1-1024x646.png" alt="" class="wp-image-772" width="512" height="323" /><figcaption>Nginx版本对应的补丁</figcaption></figure>
</div>

先安装打补丁的工具：

<pre class="wp-block-preformatted">sudo yum install patch</pre>

再打补丁：

<pre class="wp-block-preformatted"># 进入Nginx目录下<br />cd ~/nginx-1.18.0<br /># 打补丁<br />patch -p1 &lt; /home/robin/ngx_http_proxy_connect_module/patch/proxy_connect_rewrite_1018.patch</pre>

在Nginx目录下执行如下命令：

<pre class="wp-block-preformatted"># 进入Nginx目录下<br />cd ~/nginx-1.18.0<br /># 配置Nginx的目录和模块<br />./configure \ <br />--user=www \ <br />--group=www \ <br />--prefix=/usr/local/nginx \ <br />--with-http_ssl_module \ <br />--with-http_stub_status_module \ <br />--with-http_realip_module \ <br />--with-threads \ <br />--add-module=/home/robin/ngx_http_proxy_connect_module<br /># 编译<br />sudo make<br /># 安装<br />sudo make install<br /></pre>

在/usr/local/nginx/conf/nginx.conf的http块中加入下面一段server块：

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2021/02/image-2-1024x544.png" alt="" class="wp-image-773" width="512" height="272" /><figcaption>http块中的server块<br /></figcaption></figure>
</div>

代码如下：

<pre class="wp-block-preformatted">server {<br />    listen  443;<br /><code>&nbsp;   # dns resolver used by forward proxying  </code><br /><code>&nbsp;   resolver  8.8.8.8;  </code><br /><code>&nbsp;   </code><br /><code>&nbsp;   # forward proxy for CONNECT request  </code><br /><code>&nbsp;   proxy_connect;&nbsp;</code><br /><code>&nbsp;   proxy_connect_allow            443;  </code><br /><code>&nbsp;   proxy_connect_connect_timeout  10s;  </code><br /><code>&nbsp;   proxy_connect_read_timeout     10s;  </code><br /><code>&nbsp;   proxy_connect_send_timeout     10s;  </code><br /><code>&nbsp;   # forward proxy for non-CONNECT request  </code><br /><code>&nbsp;   location / {      </code><br /><code>&nbsp;       proxy_pass http://$host;      </code><br /><code>&nbsp;       proxy_set_header Host $host;  </code><br /><code>&nbsp;   }</code><br /> }</pre>

启动Nginx：

<pre class="wp-block-preformatted"># 保存退出<br />:wq<br /># 启动Nginx<br />cd /usr/local/nginx/sbin/ && sudo ./nginx</pre>

在我的macbook终端做测试（\*.\*.\*.\*为我的云服务器外网IP）：

<pre class="wp-block-preformatted">curl https://www.baidu.com -svo /dev/null -x *.*.*.*:443<br /> About to connect() to proxy *.*.*.* port 443 (#0)<br /> Trying *.*.*.*…<br /> Connected to *.*.*.* (*.*.*.*) port 443 (#0)<br /> Establish HTTP proxy tunnel to www.baidu.com:443 <br />   CONNECT www.baidu.com:443 HTTP/1.1<br />   Host: www.baidu.com:443<br />   User-Agent: curl/7.29.0<br />   Proxy-Connection: Keep-Alive<br />   &lt; HTTP/1.1 200 Connection Established<br />   &lt; Proxy-agent: nginx<br />   &lt;<br />      Proxy replied OK to CONNECT request<br />   Initializing NSS with certpath: sql:/etc/pki/nssdb<br />   CAfile: /etc/pki/tls/certs/ca-bundle.crt<br />   CApath: none<br />   SSL connection using TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256<br />   Server certificate:<br />   subject: CN=baidu.com,O="Beijing Baidu Netcom Science Technology Co., Ltd",OU=service operation department,L=beijing,ST=beijing,C=CN<br />   …<br />   GET / HTTP/1.1<br />   User-Agent: curl/7.29.0<br />   Host: www.baidu.com<br />   Accept: <em>/</em>   <br />   &lt; HTTP/1.1 200 OK<br />   …<br />   { [data not shown] </pre>

从上面-v参数打印出的细节，可以看到客户端先往代理服务器\*.\*.\*.\*建立了HTTP CONNECT隧道，代理回复HTTP/1.1 200 Connection Established后就开始交互TLS/SSL握手和流量了。  


下面以谷歌浏览器插件配置代理服务器为例，其他浏览器请读者自行查找配置代理服务器的方法。

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2021/02/image-3-1024x318.png" alt="" class="wp-image-774" width="512" height="159" /><figcaption>代理服务器为我的云服务器外网IP<br /></figcaption></figure>
</div>

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2021/02/image-4-1024x437.png" alt="" class="wp-image-775" width="512" height="219" /><figcaption>启动代理</figcaption></figure>
</div>

至此，我的朋友已经可以在他的笔记本通过我搭的正向代理服务器访问各种墙内的网站了。至此，读者可能以为问题被解决了，但，B站依旧访问不到。所以B站网址应该在更底层的网络层被限制了。所以看到这的读者就当作增加一些奇怪的知识吧 😀

文章参考来源[「使用NGINX作为HTTPS正向代理服务器」][2]

 [1]: https://github.com/chobits/ngx_http_proxy_connect_module
 [2]: https://developer.aliyun.com/article/706196