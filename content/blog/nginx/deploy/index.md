---
title: Nginx
date: 2020-12-09T10:28:38+00:00
categories:
  - Nginx
---
本文主要介绍以下几点，关于Nginx的部署，读者可自行参考其他资料。

  * Nginx核心配置文件解读
  * Nginx应用场景之反向代理
  * Nginx应用场景之负载均衡
  * Nginx应用场景之动静分离
  * Nginx底层进程机制解剖

<div id="ez-toc-container" class="ez-toc-v2_0_56_1 counter-hierarchy ez-toc-counter ez-toc-grey ez-toc-container-direction">
  <div class="ez-toc-title-container">
    <p class="ez-toc-title " >
      目录
    </p>
    
    <span class="ez-toc-title-toggle"><a href="#" class="ez-toc-pull-right ez-toc-btn ez-toc-btn-xs ez-toc-btn-default ez-toc-toggle" aria-label="Toggle Table of Content" role="button"><label for="item-6896f659ae77e" ><span class=""><span style="display:none;">Toggle</span><span class="ez-toc-icon-toggle-span"><svg style="fill: #999;color:#999" xmlns="http://www.w3.org/2000/svg" class="list-377408" width="20px" height="20px" viewBox="0 0 24 24" fill="none"><path d="M6 6H4v2h2V6zm14 0H8v2h12V6zM4 11h2v2H4v-2zm16 0H8v2h12v-2zM4 16h2v2H4v-2zm16 0H8v2h12v-2z" fill="currentColor"></path></svg><svg style="fill: #999;color:#999" class="arrow-unsorted-368013" xmlns="http://www.w3.org/2000/svg" width="10px" height="10px" viewBox="0 0 24 24" version="1.2" baseProfile="tiny"><path d="M18.2 9.3l-6.2-6.3-6.2 6.3c-.2.2-.3.4-.3.7s.1.5.3.7c.2.2.4.3.7.3h11c.3 0 .5-.1.7-.3.2-.2.3-.5.3-.7s-.1-.5-.3-.7zM5.8 14.7l6.2 6.3 6.2-6.3c.2-.2.3-.5.3-.7s-.1-.5-.3-.7c-.2-.2-.4-.3-.7-.3h-11c-.3 0-.5.1-.7.3-.2.2-.3.5-.3.7s.1.5.3.7z"/></svg></span></span></label><input aria-label="Toggle" aria-label="item-6896f659ae77e"  type="checkbox" id="item-6896f659ae77e" /></a></span>
  </div><nav>
  
  <ul class='ez-toc-list ez-toc-list-level-1 ' >
    <li class='ez-toc-page-1 ez-toc-heading-level-2'>
      <a class="ez-toc-link ez-toc-heading-1" href="http://43.133.160.241/?p=698/#1_Nginx%E6%A0%B8%E5%BF%83%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E8%A7%A3%E8%AF%BB" title="1. Nginx核心配置文件解读">1. Nginx核心配置文件解读</a><ul class='ez-toc-list-level-3'>
        <li class='ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-2" href="http://43.133.160.241/?p=698/#11_%E5%85%A8%E5%B1%80%E9%83%A8%E5%88%86" title="1.1. 全局部分">1.1. 全局部分</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-3" href="http://43.133.160.241/?p=698/#12_events%E9%83%A8%E5%88%86" title="1.2.&nbsp;events部分">1.2.&nbsp;events部分</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-4" href="http://43.133.160.241/?p=698/#13_http%E9%83%A8%E5%88%86" title="1.3.&nbsp;http部分">1.3.&nbsp;http部分</a>
        </li>
      </ul>
    </li>
    
    <li class='ez-toc-page-1 ez-toc-heading-level-2'>
      <a class="ez-toc-link ez-toc-heading-5" href="http://43.133.160.241/?p=698/#2_Nginx%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF%E4%B9%8B%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86" title="2.&nbsp;Nginx应用场景之反向代理">2.&nbsp;Nginx应用场景之反向代理</a><ul class='ez-toc-list-level-3'>
        <li class='ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-6" href="http://43.133.160.241/?p=698/#21_%E5%9C%BA%E6%99%AF1" title="2.1.&nbsp;场景1">2.1.&nbsp;场景1</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-7" href="http://43.133.160.241/?p=698/#22_%E5%9C%BA%E6%99%AF2" title="2.2.&nbsp;场景2">2.2.&nbsp;场景2</a>
        </li>
      </ul>
    </li>
    
    <li class='ez-toc-page-1 ez-toc-heading-level-2'>
      <a class="ez-toc-link ez-toc-heading-8" href="http://43.133.160.241/?p=698/#3_Nginx%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF%E4%B9%8B%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1" title="3.&nbsp;Nginx应用场景之负载均衡">3.&nbsp;Nginx应用场景之负载均衡</a>
    </li>
    <li class='ez-toc-page-1 ez-toc-heading-level-2'>
      <a class="ez-toc-link ez-toc-heading-9" href="http://43.133.160.241/?p=698/#4_Nginx%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF%E4%B9%8B%E5%8A%A8%E9%9D%99%E5%88%86%E7%A6%BB" title="4.&nbsp;Nginx应用场景之动静分离">4.&nbsp;Nginx应用场景之动静分离</a>
    </li>
    <li class='ez-toc-page-1 ez-toc-heading-level-2'>
      <a class="ez-toc-link ez-toc-heading-10" href="http://43.133.160.241/?p=698/#5_Nginx%E5%BA%95%E5%B1%82%E8%BF%9B%E7%A8%8B%E6%9C%BA%E5%88%B6%E5%89%96%E6%9E%90" title="5.&nbsp;Nginx底层进程机制剖析">5.&nbsp;Nginx底层进程机制剖析</a>
    </li>
  </ul></nav>
</div>

## <span class="ez-toc-section" id="1_Nginx%E6%A0%B8%E5%BF%83%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E8%A7%A3%E8%AF%BB"></span>1. Nginx核心配置文件解读<span class="ez-toc-section-end"></span> {.wp-block-heading}

Nginx的核心配置文件conf/nginx.conf包含三个部分内容：全局部分、events部分、http部分。

### <span class="ez-toc-section" id="11_%E5%85%A8%E5%B1%80%E9%83%A8%E5%88%86"></span>**1.1. 全局部分**<span class="ez-toc-section-end"></span> {.wp-block-heading}

从配置文件开始到events块之间的内容，此处的配置影响nginx服务器整体的运行，比如worker进程的数量、错误日志的位置等。<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/12/image-1-1024x297.png" alt="" class="wp-image-700" /> <figcaption>图1 全局部分</figcaption></figure> 

### <span class="ez-toc-section" id="12_events%E9%83%A8%E5%88%86"></span>**1.2.&nbsp;events部分**<span class="ez-toc-section-end"></span> {.wp-block-heading}

events块主要影响Nginx服务器与用户的网络连接，比如worker_connections 1024，标识每个worker进程支持的最大链接数为1024。<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/12/image-2-1024x128.png" alt="" class="wp-image-701" /> <figcaption>图2 events部分</figcaption></figure> 

### <span class="ez-toc-section" id="13_http%E9%83%A8%E5%88%86"></span>**1.3.&nbsp;http部分**<span class="ez-toc-section-end"></span> {.wp-block-heading}

http部分是配置最频繁的部分，虚拟主机的配置，监听端口的配置，请求转发、反向代理、负载均衡等。<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/12/image-6-770x1024.png" alt="" class="wp-image-705" /> <figcaption>图3 http部分</figcaption></figure> <figure class="wp-block-image"><img decoding="async" src="http://roliu.work/wp-content/uploads/2020/12/image-7-786x1024.png" alt="" class="wp-image-706" /><figcaption>图4 http部分</figcaption></figure> 

## <span class="ez-toc-section" id="2_Nginx%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF%E4%B9%8B%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86"></span>**2.&nbsp;Nginx应用场景之反向代理**<span class="ez-toc-section-end"></span> {.wp-block-heading}

### <span class="ez-toc-section" id="21_%E5%9C%BA%E6%99%AF1"></span>**2.1.&nbsp;场景1**<span class="ez-toc-section-end"></span> {.wp-block-heading}

**需求：**

浏览器请求Nginx服务器，Nginx将请求转发给一个目标服务器，用户看到的是目标服务器的相应页面。整个过程中目标服务器对客户端不可见，服务端向外暴露的是Nginx的地址。

**实现：**

  * 部署Tomcat，保持默认监听8080端口；
  * 修改Nginx配置，并重新加载；
      * 修改Nginx配置，如图5
      * 重新加载Nginx配置（./nginx -s reload）
  * 测试，访问http://[Nginx IP]:9003，返回Tomcat的页面<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/12/image-8-1024x559.png" alt="" class="wp-image-707" /> <figcaption>图5 修改后的Nginx配置</figcaption></figure> 

### <span class="ez-toc-section" id="22_%E5%9C%BA%E6%99%AF2"></span>**2.2.&nbsp;场景2**<span class="ez-toc-section-end"></span> {#mce_45.wp-block-heading}

**需求：**

在场景1的基础上，目标服务器有两个，分别是127.0.0.1:8080，127.0.0.1:8081  
当访问http://[Nginx IP]:9003/a的时候，实际访问目标服务器127.0.0.1:8080  
当访问http://[Nginx IP]:9003/b的时候，实际访问目标服务器127.0.0.1:8081

**实现：**

  * 再部署一个Tomcat，并保持默认监听8081端口；
  * 修改Nginx配置，并重新加载
      * 修改Nginx配置，如图6
      * 重新加载Nginx配置（./nginx -s reload）<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/12/image-10-1024x543.png" alt="" class="wp-image-709" /> <figcaption>图6 修改后的Nginx配置</figcaption></figure> 

<p class="has-luminous-vivid-orange-color has-text-color">
  *若不在端口号后面加上/，Nginx会转发请求至http://127.0.0.1:8080/a<br />加上/，Nginx会转发请求至http://127.0.0.1:8080
</p>

location语法

<pre>location [=|~|~*|^~] /uri/ {...}</pre>

在Nginx配置文件中，location主要有这几种形式：

  1. 正则表达式 location ~/a {&#8230;}
  2. 不区分大小写的正则匹配 location ~*/a {&#8230;}\
  3. 匹配路径的前缀 location ^~/a {&#8230;}
  4. 精确匹配 location = /a {&#8230;}
  5. 普通路径前缀匹配 location /a {&#8230;}

优先级：4 > 3 > 2 > 1 >5

## <span class="ez-toc-section" id="3_Nginx%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF%E4%B9%8B%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1"></span>**3.&nbsp;Nginx应用场景之负载均衡**<span class="ez-toc-section-end"></span> {#mce_41.wp-block-heading}

Nginx负载均衡策略有以下三个：

  * 轮询
  * weight
  * ip_hash

**3.1.&nbsp;轮询**

默认策略，每个请求按时间顺序逐一分配到不同的服务器，如果某一个服务器下线，能自动剔除。

<pre>upstream robinServer{<br /> server [Nginx IP]:8080;<br /> server [Nginx IP]:8082;<br /> }<br /><br /> location /abc {<br /> proxy_pass http://robinServer/;<br /> } </pre>

**3.2.&nbsp;weight**

weight代表权重，默认每一个负载的服务器都为1，权重越高被分配的请求越多。（可用于服务器性能不均衡的场景）

<pre>upstream robinServer{<br /> server [Nginx IP]:8080 weight=1;<br /> server [Nginx IP]:8082 weight=2;<br /> } </pre>

**3.3.&nbsp;ip_hash**

每个请求按照ip的hash结果分配，每一个客户端的请求会固定分配到同一个目标服务器处理，可以解决session问题。

<pre>upstream robinServer{<br /> ip_hash;<br /> server [Nginx IP]:8080;<br /> server [Nginx IP]:8082;<br /> } </pre>

## <span class="ez-toc-section" id="4_Nginx%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF%E4%B9%8B%E5%8A%A8%E9%9D%99%E5%88%86%E7%A6%BB"></span>**4.&nbsp;Nginx应用场景之动静分离**<span class="ez-toc-section-end"></span> {#mce_87.wp-block-heading}

动静分离就是讲动态资源和静态资源的请求处理分配到不同的服务器上，比较经典的组合就是Nginx+Tomcat架构。（Nginx处理静态资源请求，Tomcat处理动态资源请求）<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/12/Nginx-backend2-1-1024x520.png" alt="" class="wp-image-712" /> <figcaption>图7 简单的动静分离架构  
</figcaption></figure> 

**Nginx配置**<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/12/image-15.png" alt="" class="wp-image-719" /> <figcaption>图8 Nginx配置</figcaption></figure> 

实际在生产建议可以专门弄个静态资源服务器，架构如下：<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/12/Nginx-backend3-1024x621.png" alt="" class="wp-image-713" /> <figcaption>图9 动静分离架构</figcaption></figure> 

## <span class="ez-toc-section" id="5_Nginx%E5%BA%95%E5%B1%82%E8%BF%9B%E7%A8%8B%E6%9C%BA%E5%88%B6%E5%89%96%E6%9E%90"></span>**5.&nbsp;Nginx底层进程机制剖析**<span class="ez-toc-section-end"></span> {#mce_106.wp-block-heading}

Nginx启动后，以Daemon多进程方式在后台运行，包括一个Master进程和多个Worker进程。<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/12/image-14-1024x80.png" alt="" class="wp-image-716" /> <figcaption>图10 Nginx进程  
</figcaption></figure> 

**Master进程**  
管理Worker进程，比如：

  * 接收外界信号向各Worker进程发送信号（./nginx -s reload）
  * 监控Worker进程的运行状态，当Worker进程异常推出后Master进程会自动重新启动新的Worker进程等等

**Worker进程**

Worker进程具体处理网络请求，多个Worker进程之间是对等的，他们同等竞争来自客户端的请求，**各进程相互之间是独立的**。一个请求只能在一个Worker进程中处理，一个Worker进程不可能处理其他进程的请求。Worker进程的个数是可以设置的，一般设置与服务器CPU核数一致。<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/12/nginx进程-1-1024x689.png" alt="" class="wp-image-718" /> <figcaption>图11 Nginx进程模型</figcaption></figure> 

以 ./nginx -s reload来说明nginx信号处理这部分

  1. master进程对配置文件进行语法检查；
  2. 尝试配置（比如修改了监听端口，就会尝试分配新的监听端口）；
  3. 尝试成功则使用新的配置，新建Worker进程；
  4. 新建成功，给旧的Worker进程发送关闭消息；
  5. 旧的Worker进程收到信号会继续服务，知道把当前进程收到的请求处理完毕

所以reload后Worker进程pid发生了变化<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/12/image-17-1024x226.png" alt="" class="wp-image-721" /> <figcaption>图12 reload nginx配置文件  
</figcaption></figure> 

**Worker进程请求处理部分说明**

例如，我们监听9003端口，一个请求到来时，如果有多个Worker进程，那么每个Worker进程都有可能处理这个链接。

  * Master进程创建之后，会建立好需要监听的socket，然后Master进程再fork出多个Worker进程。所以，所有Worker进程的监听描述符linstenfd在新链接到来时都变得可读；
  * nginx使用互斥锁来保证只有一个Worker进程能处理请求，拿到互斥锁的那个进程注册listenfd读事件，在读事件里调用accept接受该链接，然后解析、处理、返回客户端。

Nginx多进程模型好处

  * 每个Worker进程都是独立的，不需要加锁，节省开销；
  * 每个Worker进程都是独立的，互不影响，其中一个进程异常导致其结束，其他的照样能提供服务；
  * 多进程模型为reload热部署机制提供了支撑。