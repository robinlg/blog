---
title: Apache Tomcat Web Application Server
date: 2020-11-07T07:32:01+00:00
categories:
  - Linux
---
此文介绍Tomcat相关原理及配置，文章并非一次将所有关于Tomcat（基于8.5.50）的知识点全罗列出来，因此文章可能会不定期更新。若有更新会及时更新以下的目录：

  1. **Tomcat系统架构与原理剖析**
  2. **Tomcat服务器核心配置**
  3. **Tomcat类加载机制剖析**
  4. **Tomcat对HTTPS的支持及Tomcat性能优化策略**

<div id="ez-toc-container" class="ez-toc-v2_0_56_1 counter-hierarchy ez-toc-counter ez-toc-grey ez-toc-container-direction">
  <div class="ez-toc-title-container">
    <p class="ez-toc-title " >
      目录
    </p>
    
    <span class="ez-toc-title-toggle"><a href="#" class="ez-toc-pull-right ez-toc-btn ez-toc-btn-xs ez-toc-btn-default ez-toc-toggle" aria-label="Toggle Table of Content" role="button"><label for="item-6896f6597b312" ><span class=""><span style="display:none;">Toggle</span><span class="ez-toc-icon-toggle-span"><svg style="fill: #999;color:#999" xmlns="http://www.w3.org/2000/svg" class="list-377408" width="20px" height="20px" viewBox="0 0 24 24" fill="none"><path d="M6 6H4v2h2V6zm14 0H8v2h12V6zM4 11h2v2H4v-2zm16 0H8v2h12v-2zM4 16h2v2H4v-2zm16 0H8v2h12v-2z" fill="currentColor"></path></svg><svg style="fill: #999;color:#999" class="arrow-unsorted-368013" xmlns="http://www.w3.org/2000/svg" width="10px" height="10px" viewBox="0 0 24 24" version="1.2" baseProfile="tiny"><path d="M18.2 9.3l-6.2-6.3-6.2 6.3c-.2.2-.3.4-.3.7s.1.5.3.7c.2.2.4.3.7.3h11c.3 0 .5-.1.7-.3.2-.2.3-.5.3-.7s-.1-.5-.3-.7zM5.8 14.7l6.2 6.3 6.2-6.3c.2-.2.3-.5.3-.7s-.1-.5-.3-.7c-.2-.2-.4-.3-.7-.3h-11c-.3 0-.5.1-.7.3-.2.2-.3.5-.3.7s.1.5.3.7z"/></svg></span></span></label><input aria-label="Toggle" aria-label="item-6896f6597b312"  type="checkbox" id="item-6896f6597b312" /></a></span>
  </div><nav>
  
  <ul class='ez-toc-list ez-toc-list-level-1 ' >
    <li class='ez-toc-page-1 ez-toc-heading-level-2'>
      <a class="ez-toc-link ez-toc-heading-1" href="http://43.133.160.241/?p=641/#1_Tomcat%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84%E4%B8%8E%E5%8E%9F%E7%90%86%E5%89%96%E6%9E%90" title="1. Tomcat系统架构与原理剖析">1. Tomcat系统架构与原理剖析</a><ul class='ez-toc-list-level-3'>
        <li class='ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-2" href="http://43.133.160.241/?p=641/#11_%E6%B5%8F%E8%A7%88%E5%99%A8%E8%AE%BF%E9%97%AE%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9A%84%E6%B5%81%E7%A8%8B" title="1.1. 浏览器访问服务器的流程">1.1. 浏览器访问服务器的流程</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-3" href="http://43.133.160.241/?p=641/#12_Tomcat%E7%B3%BB%E7%BB%9F%E6%80%BB%E4%BD%93%E6%9E%B6%E6%9E%84" title="1.2.&nbsp;Tomcat系统总体架构">1.2.&nbsp;Tomcat系统总体架构</a><ul class='ez-toc-list-level-4'>
            <li class='ez-toc-heading-level-4'>
              <a class="ez-toc-link ez-toc-heading-4" href="http://43.133.160.241/?p=641/#121_Tomcat%E8%AF%B7%E6%B1%82%E5%A4%84%E7%90%86%E5%A4%A7%E8%87%B4%E8%BF%87%E7%A8%8B" title="1.2.1. Tomcat请求处理大致过程">1.2.1. Tomcat请求处理大致过程</a>
            </li>
            <li class='ez-toc-page-1 ez-toc-heading-level-4'>
              <a class="ez-toc-link ez-toc-heading-5" href="http://43.133.160.241/?p=641/#122_Tomcat_Servlet%E5%AE%B9%E5%99%A8%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B" title="1.2.2. Tomcat&nbsp;Servlet容器处理流程">1.2.2. Tomcat&nbsp;Servlet容器处理流程</a>
            </li>
            <li class='ez-toc-page-1 ez-toc-heading-level-4'>
              <a class="ez-toc-link ez-toc-heading-6" href="http://43.133.160.241/?p=641/#123_Tomcat%E7%B3%BB%E7%BB%9F%E6%80%BB%E4%BD%93%E6%9E%B6%E6%9E%84" title="1.2.3. Tomcat系统总体架构">1.2.3. Tomcat系统总体架构</a>
            </li>
          </ul>
        </li>
        
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-7" href="http://43.133.160.241/?p=641/#13_Tomcat%E8%BF%9E%E6%8E%A5%E5%99%A8%E7%BB%84%E4%BB%B6Coyote" title="1.3.&nbsp;Tomcat连接器组件Coyote">1.3.&nbsp;Tomcat连接器组件Coyote</a><ul class='ez-toc-list-level-4'>
            <li class='ez-toc-heading-level-4'>
              <a class="ez-toc-link ez-toc-heading-8" href="http://43.133.160.241/?p=641/#131_Coyote%E7%AE%80%E4%BB%8B" title="1.3.1. Coyote简介">1.3.1. Coyote简介</a>
            </li>
            <li class='ez-toc-page-1 ez-toc-heading-level-4'>
              <a class="ez-toc-link ez-toc-heading-9" href="http://43.133.160.241/?p=641/#132_Coyote%E7%9A%84%E5%86%85%E9%83%A8%E7%BB%84%E4%BB%B6%E5%8F%8A%E6%B5%81%E7%A8%8B" title="1.3.2. Coyote的内部组件及流程">1.3.2. Coyote的内部组件及流程</a>
            </li>
          </ul>
        </li>
        
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-10" href="http://43.133.160.241/?p=641/#14_Tomcat_Servlet%E5%AE%B9%E5%99%A8Catalina" title="1.4.&nbsp;Tomcat Servlet容器Catalina">1.4.&nbsp;Tomcat Servlet容器Catalina</a><ul class='ez-toc-list-level-4'>
            <li class='ez-toc-heading-level-4'>
              <a class="ez-toc-link ez-toc-heading-11" href="http://43.133.160.241/?p=641/#141_Tomcat%E6%A8%A1%E5%9D%97%E5%88%86%E5%B1%82%E7%BB%93%E6%9E%84%E5%9B%BE%E5%8F%8ACatalina%E4%BD%8D%E7%BD%AE" title="1.4.1. Tomcat模块分层结构图及Catalina位置">1.4.1. Tomcat模块分层结构图及Catalina位置</a>
            </li>
            <li class='ez-toc-page-1 ez-toc-heading-level-4'>
              <a class="ez-toc-link ez-toc-heading-12" href="http://43.133.160.241/?p=641/#142_Servlet%E5%AE%B9%E5%99%A8Catalina%E7%9A%84%E7%BB%93%E6%9E%84" title="1.4.2. Servlet容器Catalina的结构">1.4.2. Servlet容器Catalina的结构</a>
            </li>
            <li class='ez-toc-page-1 ez-toc-heading-level-4'>
              <a class="ez-toc-link ez-toc-heading-13" href="http://43.133.160.241/?p=641/#143_Container%E7%BB%84%E4%BB%B6%E7%9A%84%E5%85%B7%E4%BD%93%E7%BB%93%E6%9E%84" title="1.4.3. Container组件的具体结构">1.4.3. Container组件的具体结构</a>
            </li>
          </ul>
        </li>
      </ul>
    </li>
    
    <li class='ez-toc-page-1 ez-toc-heading-level-2'>
      <a class="ez-toc-link ez-toc-heading-14" href="http://43.133.160.241/?p=641/#2_Tomcat%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%A0%B8%E5%BF%83%E9%85%8D%E7%BD%AE" title="2. Tomcat服务器核心配置">2. Tomcat服务器核心配置</a>
    </li>
    <li class='ez-toc-page-1 ez-toc-heading-level-2'>
      <a class="ez-toc-link ez-toc-heading-15" href="http://43.133.160.241/?p=641/#3_Tomcat%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6%E5%89%96%E6%9E%90" title="3. Tomcat类加载机制剖析">3. Tomcat类加载机制剖析</a><ul class='ez-toc-list-level-3'>
        <li class='ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-16" href="http://43.133.160.241/?p=641/#31_JVM%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6" title="3.1.&nbsp;JVM的类加载机制">3.1.&nbsp;JVM的类加载机制</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-17" href="http://43.133.160.241/?p=641/#32_%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%9C%BA%E5%88%B6" title="3.2.&nbsp;双亲委派机制">3.2.&nbsp;双亲委派机制</a><ul class='ez-toc-list-level-4'>
            <li class='ez-toc-heading-level-4'>
              <a class="ez-toc-link ez-toc-heading-18" href="http://43.133.160.241/?p=641/#321_%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%9C%BA%E5%88%B6%E7%AE%80%E8%BF%B0" title="3.2.1. 双亲委派机制简述">3.2.1. 双亲委派机制简述</a>
            </li>
            <li class='ez-toc-page-1 ez-toc-heading-level-4'>
              <a class="ez-toc-link ez-toc-heading-19" href="http://43.133.160.241/?p=641/#322_%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%9C%BA%E5%88%B6%E7%9A%84%E4%BD%9C%E7%94%A8" title="3.2.2. 双亲委派机制的作用">3.2.2. 双亲委派机制的作用</a>
            </li>
          </ul>
        </li>
        
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-20" href="http://43.133.160.241/?p=641/#33_Tomcat%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6" title="3.3.&nbsp;Tomcat的类加载机制">3.3.&nbsp;Tomcat的类加载机制</a>
        </li>
      </ul>
    </li>
    
    <li class='ez-toc-page-1 ez-toc-heading-level-2'>
      <a class="ez-toc-link ez-toc-heading-21" href="http://43.133.160.241/?p=641/#4_Tomcat%E5%AF%B9HTTPS%E7%9A%84%E6%94%AF%E6%8C%81%E5%8F%8ATomcat%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%E7%AD%96%E7%95%A5" title="4. Tomcat对HTTPS的支持及Tomcat性能优化策略">4. Tomcat对HTTPS的支持及Tomcat性能优化策略</a><ul class='ez-toc-list-level-3'>
        <li class='ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-22" href="http://43.133.160.241/?p=641/#41_Tomcat%E5%AF%B9HTTPS%E7%9A%84%E6%94%AF%E6%8C%81" title="4.1.&nbsp;Tomcat对HTTPS的支持">4.1.&nbsp;Tomcat对HTTPS的支持</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-23" href="http://43.133.160.241/?p=641/#42_Tomcat%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%E7%AD%96%E7%95%A5" title="4.2.&nbsp;Tomcat性能优化策略">4.2.&nbsp;Tomcat性能优化策略</a><ul class='ez-toc-list-level-4'>
            <li class='ez-toc-heading-level-4'>
              <a class="ez-toc-link ez-toc-heading-24" href="http://43.133.160.241/?p=641/#421_%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%BF%90%E8%A1%8C%E4%BC%98%E5%8C%96%EF%BC%88%E5%8F%82%E6%95%B0%E8%B0%83%E6%95%B4%EF%BC%89" title="4.2.1. 虚拟机运行优化（参数调整）">4.2.1. 虚拟机运行优化（参数调整）</a>
            </li>
            <li class='ez-toc-page-1 ez-toc-heading-level-4'>
              <a class="ez-toc-link ez-toc-heading-25" href="http://43.133.160.241/?p=641/#4211_JVM%E5%86%85%E5%AD%98%E7%9B%B8%E5%85%B3%E5%8F%82%E6%95%B0" title="4.2.1.1. JVM内存相关参数">4.2.1.1. JVM内存相关参数</a>
            </li>
            <li class='ez-toc-page-1 ez-toc-heading-level-4'>
              <a class="ez-toc-link ez-toc-heading-26" href="http://43.133.160.241/?p=641/#4212_%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%EF%BC%88GC%EF%BC%89%E7%AD%96%E7%95%A5" title="4.2.1.2. 垃圾回收（GC）策略">4.2.1.2. 垃圾回收（GC）策略</a>
            </li>
            <li class='ez-toc-page-1 ez-toc-heading-level-4'>
              <a class="ez-toc-link ez-toc-heading-27" href="http://43.133.160.241/?p=641/#422_Tomcat%E9%85%8D%E7%BD%AE%E8%B0%83%E4%BC%98" title="4.2.2. Tomcat配置调优">4.2.2. Tomcat配置调优</a>
            </li>
          </ul>
        </li>
      </ul>
    </li>
  </ul></nav>
</div>

## <span class="ez-toc-section" id="1_Tomcat%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84%E4%B8%8E%E5%8E%9F%E7%90%86%E5%89%96%E6%9E%90"></span>**1. Tomcat系统架构与原理剖析**<span class="ez-toc-section-end"></span> {.wp-block-heading}

### <span class="ez-toc-section" id="11_%E6%B5%8F%E8%A7%88%E5%99%A8%E8%AE%BF%E9%97%AE%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9A%84%E6%B5%81%E7%A8%8B"></span>**1.1. 浏览器访问服务器的流程**<span class="ez-toc-section-end"></span> {.wp-block-heading}

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/浏览器访问服务器流程-1024x875.png" alt="" class="wp-image-645" /><figcaption>图1 HTTP请求的处理过程</figcaption></figure>
</div>

  * HTTP请求只是定义数据的组织格式（通信格式），是应用层协议。数据传输依靠的是TCP/IP协议；
  * Tomcat使用socket实现了浏览器与服务器的通信；
  * <figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/TCP报文格式.png" alt="" class="wp-image-646" /> <figcaption>图2 TCP报文格式</figcaption></figure> 

### <span class="ez-toc-section" id="12_Tomcat%E7%B3%BB%E7%BB%9F%E6%80%BB%E4%BD%93%E6%9E%B6%E6%9E%84"></span>**1.2.&nbsp;Tomcat系统总体架构**<span class="ez-toc-section-end"></span> {.wp-block-heading}

#### <span class="ez-toc-section" id="121_Tomcat%E8%AF%B7%E6%B1%82%E5%A4%84%E7%90%86%E5%A4%A7%E8%87%B4%E8%BF%87%E7%A8%8B"></span>**1.2.1. Tomcat请求处理大致过程**<span class="ez-toc-section-end"></span> {.wp-block-heading}

Tomcat是HTTP服务器，Tomcat接收到浏览器发送的HTTP请求后，会调用相应的业务类。如图3：<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/HTTP服务器1.png" alt="" class="wp-image-647" /> <figcaption>图3 HTTP服务器调用业务类</figcaption></figure> 

如图3，若HTTP服务器（Tomcat）直接调用业务类完成业务处理，那么会存在耦合性大的问题。所以实际上Tomcat的结构如图4:<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/HTTP服务器2.png" alt="" class="wp-image-648" /> <figcaption>图4 Servlet容器调用业务类</figcaption></figure> 

HTTP服务器接收请求后分配给Servlet容器处理，Servlet容器通过Servlet接口调用业务类。Servlet接口和Servlet容器这一整套内容叫作Servlet规范。  
* Tomcat既按照Servlet规范的要求实现了Servlet容器， 同时也具有HTTP服务器的功能。因此Tomcat的两个重要部分：1）HTTP服务器；2）Servlet容器

#### <span class="ez-toc-section" id="122_Tomcat_Servlet%E5%AE%B9%E5%99%A8%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B"></span>**1.2.2. Tomcat&nbsp;Servlet容器处理流程**<span class="ez-toc-section-end"></span> {.wp-block-heading}<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/Servlet容器处理流程.png" alt="" class="wp-image-649" /> <figcaption>图5 Tomcat Servlet容器处理流程</figcaption></figure> 

  1. HTTP服务器将请求信息使用ServletRequest对象封装起来；
  2. Servlet容器拿到请求后，根据URL和Servlet的映射关系，找到相应的Servlet；
  3. 如果Servlet还没被加载，就用反射机制创建这个Servlet，并调用Servlet的init方法完成初始化；
  4. 调用此Servlet的service方法来处理请求，处理结果使用ServletResponse对象封装；
  5. 将ServletResponse对象返回给HTTP服务器，HTTP服务器会将响应发送给客户端。

#### <span class="ez-toc-section" id="123_Tomcat%E7%B3%BB%E7%BB%9F%E6%80%BB%E4%BD%93%E6%9E%B6%E6%9E%84"></span>**1.2.3. Tomcat系统总体架构**<span class="ez-toc-section-end"></span> {.wp-block-heading}<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/Tomcat总体结构.png" alt="" class="wp-image-650" /> <figcaption>图6 Tomcat两个核心组件</figcaption></figure> 

Tomcat设计了两个核心组件**连接器（Connector）**和**容器（Container）**来实现Tomcat的两大核心功能。

  * 连接器组件：负责对外交互，处理Socket链接，负责网络字节流与Request/Response对象的转换；
  * 容器组件：负责内部处理，加载和管理Servlet，处理Request请求。

### <span class="ez-toc-section" id="13_Tomcat%E8%BF%9E%E6%8E%A5%E5%99%A8%E7%BB%84%E4%BB%B6Coyote"></span>**1.3.&nbsp;Tomcat连接器组件Coyote**<span class="ez-toc-section-end"></span> {.wp-block-heading}

#### <span class="ez-toc-section" id="131_Coyote%E7%AE%80%E4%BB%8B"></span>**1.3.1. Coyote简介**<span class="ez-toc-section-end"></span> {#mce_31.wp-block-heading}

Coyote是Tomcat中连接器的组件名称，是对外的接口。客户端通过Coyote与服务器建立链接、发送请求并接收响应。<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/Coyote结构.png" alt="" class="wp-image-651" /> <figcaption>图7 核心组件交互</figcaption></figure> 

  * Coyote封装了底层的网络通信（Socket请求及响应处理）；
  * Coyote使Catalina容器（容器组件）与具体的请求协议及IP操作方式完全解耦；
  * Coyote将Socket输入转换封装为Request对象，再进一步封装后交由Catalina容器进行处理，处理请求完成后，Catalina通过Coyote提供的Response对象将结果写入输出流；
  * Coyote负责的是**HTTP协议（应用层）**和**I/O模型「BIO/NIO」（传输层）**相关内容

Tomcat Coyote支持多种应用层协议和IO模型：

  * 应用层  
      * 应用层协议 —— HTTP/1.1（默认协议）：大部分Web应用采用的访问协议；
      * 应用层协议 —— AJP：用于和WX集成（如Apache），1以实现对静态资源的优化以及集群部署，当前支持AJP/1.3；
      * 应用层协议 —— HTTP/2：HTTP2.0大幅度提升了Web性能。下一代HTTP协议，自8.5以及9.0版本后支持。
  * 传输层
      * I/O模型 —— BIO：同步阻塞I/O；
      * I/O模型 —— NIO（默认I/O模型）：非阻塞I/O，采用Java NIO类库实现；
      * I/O模型 —— NIO2：异步I/O，采用JDK 7最新的NIO2类库实现；
      * I/O模型 —— APR：采用Apache可移植运行库实现，是C/C++编写的本地库。若选择该方案，需要单独安装APR库。

在Tomcat 8.0之前，Tomcat默认采用的I/O方式为BIO，之后改为NIO。⽆论 NIO、NIO2 还是 APR， 在性能⽅⾯均优于以往的BIO。 如果采⽤APR， 甚⾄可以达到 Apache HTTP Server 的性能。

#### <span class="ez-toc-section" id="132_Coyote%E7%9A%84%E5%86%85%E9%83%A8%E7%BB%84%E4%BB%B6%E5%8F%8A%E6%B5%81%E7%A8%8B"></span>**1.3.2. Coyote的内部组件及流程**<span class="ez-toc-section-end"></span> {#mce_51.wp-block-heading}<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/Coyote内部组件-1-1024x305.png" alt="" class="wp-image-653" /> <figcaption>图8 Coyote内部组件及流程</figcaption></figure> 

Coyote组件及作用：

  * EndPoint：EndPoint 是 Coyote 通信端点，即通信监听的接⼝，是具体Socket接收和发送的处理器，是对传输层的抽象，因此EndPoint⽤来实现TCP/IP协议；
  * Processor：Processor 是Coyote 协议处理接⼝ ，如果说EndPoint是⽤来实现TCP/IP协议的，那么Processor⽤来实现HTTP协议，Processor接收来⾃EndPoint的Socket，读取字节流解析成Tomcat Request和Response对象，并通过Adapter将其提交到容器处理，Processor是对应⽤层协议的抽象；
  * ProtocolHandler：Coyote 协议接⼝， 通过Endpoint 和 Processor ， 实现针对具体协议的处理能⼒。Tomcat 按照协议和I/O 提供了6个实现类 ： AjpNioProtocol, AjpAprProtocol, AjpNio2Protocol, Http11NioProtocol ，Http11Nio2Protocol, Http11AprProtocol；
  * Adapter：由于协议不同，客户端发过来的请求信息也不尽相同，Tomcat定义了⾃⼰的Request类来封装这些请求信息。ProtocolHandler接⼝负责解析请求并⽣成Tomcat Request类。但是这个Request对象不是标准的ServletRequest，不能⽤Tomcat Request作为参数来调⽤容器。Tomcat设计者的解决⽅案是引⼊CoyoteAdapter，这是适配器模式的经典运⽤，连接器调⽤CoyoteAdapter的Sevice⽅法，传⼊的是Tomcat Request对象，CoyoteAdapter负责将Tomcat Request转成ServletRequest，再调⽤容器。

### <span class="ez-toc-section" id="14_Tomcat_Servlet%E5%AE%B9%E5%99%A8Catalina"></span>**1.4.&nbsp;Tomcat Servlet容器Catalina**<span class="ez-toc-section-end"></span> {#mce_49.wp-block-heading}

#### <span class="ez-toc-section" id="141_Tomcat%E6%A8%A1%E5%9D%97%E5%88%86%E5%B1%82%E7%BB%93%E6%9E%84%E5%9B%BE%E5%8F%8ACatalina%E4%BD%8D%E7%BD%AE"></span>**1.4.1. Tomcat模块分层结构图及Catalina位置**<span class="ez-toc-section-end"></span> {#mce_66.wp-block-heading}

Tomcat是一个由一系列可配置（conf/server.xml）的组件构成的Web容器，而Catalina是Tomcat的Servlet容器。  
换一个角度来说，Tomcat本质上就是一款Servlet容器，因为Catalina是Tomcat的核心，其他模块都是为Catalina提供支撑的。比如：通过Coyote模块提供链接通信，Jasper模块提供JSP引擎，Naming提供JNDI服务，Juli提供日志服务。<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/Catalina结构.png" alt="" class="wp-image-654" /> <figcaption>图9 Tomcat模块分层结构</figcaption></figure> 

#### <span class="ez-toc-section" id="142_Servlet%E5%AE%B9%E5%99%A8Catalina%E7%9A%84%E7%BB%93%E6%9E%84"></span>**1.4.2. Servlet容器Catalina的结构**<span class="ez-toc-section-end"></span> {#mce_101.wp-block-heading}<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/Servlet容器Catalina结构-2.png" alt="" class="wp-image-662" /> <figcaption>图10 Catalina的结构</figcaption></figure> 

其实可以认为整个Tomcat就是一个Catalina实例，Tomcat启动的时候会初始化这个实例，Catalina实例通过加载server.xml完成其他实例的创建，创建并管理一个Server，Server创建并管理多个Service，每个Service又可以有多个Connector和一个Container。  
一个Catalina实例（容器）  
一个Server实例（容器）  
多个Service实例（容器）  
每一个Service实例下可以有多个Connector实例和一个Container实例。

  * Catalina：负责解析Tomcat的配置文件（server.xml），以此创建Server组件并管理；
  * Server：表示整个Catalina Servlet容器以及其它组件，负责组装并启动Servlet引擎，Tomcat连接器。Server通过实现Lifecycle接⼝，提供了⼀种优雅的启动和关闭整个系统的⽅式；
  * Service：是Server内部的组件，⼀个Server包含多个Service。它将若⼲个Connector组件绑定到⼀个Container；
  * Container：负责处理用户的servlet请求，并返回对象给web用户的模块。

#### <span class="ez-toc-section" id="143_Container%E7%BB%84%E4%BB%B6%E7%9A%84%E5%85%B7%E4%BD%93%E7%BB%93%E6%9E%84"></span>**1.4.3. Container组件的具体结构**<span class="ez-toc-section-end"></span> {#mce_107.wp-block-heading}

Container组件下有⼏种组件，分别是Engine、Host、Context和Wrapper。这4种组件（容器）是⽗⼦关系。Tomcat通过⼀种分层的架构，使得Servlet容器具有很好的灵活性。

  * Engine：表示整个Catalina的Servlet引擎，用来管理多个虚拟站点，一个Service最多只能有一个Engine，一个Engine可以包含多个Host；
  * Host：代表一个虚拟主机，或者说一个站点，可以给Tomcat配置多个虚拟主机地址，而一个虚拟主机下可包含多个Context；
  * Context：表示一个Web应用程序，一个Web应用可包含多个Wrapper；
  * Wrapper：表示一个Servlet，Wrapper作为容器中的最底层，不能包含子容器。

## <span class="ez-toc-section" id="2_Tomcat%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%A0%B8%E5%BF%83%E9%85%8D%E7%BD%AE"></span>**2. Tomcat服务器核心配置**<span class="ez-toc-section-end"></span> {#mce_126.wp-block-heading}

  * Tomcat作为服务器的配置，主要是server.xml文件的配置；
  * server.xml中包含了Servlet容器的相关配置，即Catalina的配置。

**主要标签结构如下：**

<pre><p>
  &lt;!--<br />             Server 根元素，创建⼀个Server实例，⼦标签有 Listener、GlobalNamingResources、Service<br /> --&gt;<br /> &lt;Server&gt;<br />       &lt;!--定义监听器--&gt;<br />       &lt;Listener/&gt;<br />       &lt;!--定义服务器的全局JNDI资源 --&gt;<br />       &lt;GlobalNamingResources/&gt;<br />        &lt;!--<br />             定义⼀个Service服务，⼀个Server标签可以有多个Service服务实例<br />        --&gt;<br />       &lt;Service/&gt;<br /> &lt;/Server&gt; 
</p></pre>

**Server标签：**

<pre><p>
  &lt;!--<br />              port：关闭服务器的监听端⼝<br />              shutdown：关闭服务器的指令字符串<br /> --&gt;<br /> &lt;Server port="8005" shutdown="SHUTDOWN"&gt;<br />        &lt;!-- 以⽇志形式输出服务器 、操作系统、JVM的版本信息 --&gt;<br />        &lt;Listener className="org.apache.catalina.startup.VersionLoggerListener" /&gt;<br />        &lt;!-- Security listener. Documentation at /docs/config/listeners.html<br />        &lt;Listener className="org.apache.catalina.security.SecurityListener" /&gt;<br />        --&gt;<br />        &lt;!--APR library loader. Documentation at /docs/apr.html --&gt;<br />        &lt;!-- 加载（服务器启动） 和 销毁 （服务器停⽌） APR。 如果找不到APR库， 则会输出⽇志， 并不影响 Tomcat启动 --&gt;<br />        &lt;Listener className="org.apache.catalina.core.AprLifecycleListener"<br /> SSLEngine="on" /&gt;<br />        &lt;!-- Prevent memory leaks due to use of particular java/javax APIs--&gt;<br />        &lt;!-- 避免JRE内存泄漏问题 --&gt;<br />        &lt;Listener<br /> className="org.apache.catalina.core.JreMemoryLeakPreventionListener" /&gt;<br />        &lt;!-- 加载（服务器启动） 和 销毁（服务器停⽌） 全局命名服务 --&gt;<br />        &lt;Listener<br /> className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" /&gt;<br />        &lt;!-- 在Context停⽌时重建 Executor 池中的线程， 以避免ThreadLocal 相关的内存泄漏 --&gt; 
</p>

<p>
  &lt;Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" /&gt;<br />        &lt;!-- Global JNDI resources Documentation at /docs/jndi-resources-howto.html GlobalNamingResources 中定义了全局命名服务<br />        --&gt;<br />        &lt;GlobalNamingResources&gt;<br />        &lt;!-- Editable user database that can also be used by UserDatabaseRealm to authenticate users<br />        --&gt;<br />        &lt;Resource name="UserDatabase" auth="Container" type="org.apache.catalina.UserDatabase" description="User database that can be updated and saved" factory="org.apache.catalina.users.MemoryUserDatabaseFactory" pathname="conf/tomcat-users.xml" /&gt;<br />        &lt;/GlobalNamingResources&gt;<br />        &lt;!-- A "Service" is a collection of one or more "Connectors" that share a single "Container" Note: A "Service" is not itself a "Container", so you may not define subcomponents such as "Valves" at this level. Documentation at /docs/config/service.html --&gt;<br />        &lt;Service name="Catalina"&gt;<br />        ...<br />        &lt;/Service&gt;<br /> &lt;/Server&gt; 
</p>

<p>
  
</p></pre>

**Service标签：**

<pre><p>
  &lt;!--<br /> 该标签⽤于创建 Service 实例，默认使 org.apache.catalina.core.StandardService。<br /> 默认情况下，Tomcat 仅指定了Service 的名称， 值为 "Catalina"。<br /> Service ⼦标签为 ： Listener、Executor、Connector、Engine，<br /> 其中：<br /> Listener ⽤于为Service添加⽣命周期监听器，<br /> Executor ⽤于配置Service 共享线程池，<br /> Connector ⽤于配置Service 包含的链接器，<br /> Engine ⽤于配置Service中链接器对应的Servlet 容器引擎<br /> --&gt;<br /> &lt;Service name="Catalina"&gt;<br /> ...<br /> &lt;/Service&gt; 
</p></pre>

**Executor标签：**

<pre><p>
  &lt;!-- <br /> 默认情况下，Service 并未添加共享线程池配置。 如果我们想添加⼀个线程池， 可以在<br /> &lt;Service&gt; 下添加如下配置：<br /> name：线程池名称，⽤于 Connector中指定<br /> namePrefix：所创建的每个线程的名称前缀，⼀个单独的线程名称为<br /> namePrefix+threadNumber<br /> maxThreads：池中最⼤线程数<br /> minSpareThreads：活跃线程数，也就是核⼼池线程数，这些线程不会被销毁，会⼀直存在<br /> maxIdleTime：线程空闲时间，超过该时间后，空闲线程会被销毁，默认值为6000（1分钟），单位<br /> 毫秒<br /> maxQueueSize：在被执⾏前最⼤线程排队数⽬，默认为Int的最⼤值，也就是⼴义的⽆限。除⾮特<br /> 殊情况，这个值 不需要更改，否则会有请求不会被处理的情况发⽣<br /> prestartminSpareThreads：启动线程池时是否启动 minSpareThreads部分线程。默认值为<br /> false，即不启动<br /> threadPriority：线程池中线程优先级，默认值为5，值从1到10<br /> className：线程池实现类，未指定情况下，默认实现类为<br /> org.apache.catalina.core.StandardThreadExecutor。如果想使⽤⾃定义线程池⾸先需要实现<br /> org.apache.catalina.Executor接⼝<br /> --&gt;<br /> &lt;Executor name="commonThreadPool"<br />         namePrefix="thread-exec-"<br />         maxThreads="200"<br />         minSpareThreads="100"<br />         maxIdleTime="60000"<br />         maxQueueSize="Integer.MAX_VALUE"<br />         prestartminSpareThreads="false"<br />         threadPriority="5"<br />         className="org.apache.catalina.core.StandardThreadExecutor"/&gt; 
</p></pre>

**Connector标签：**  
大部分情况下不需要新增连接器配置，根据需要进行修改即可。

<pre><p>
  &lt;!--<br /> port：<br /> 端⼝号，Connector ⽤于创建服务端Socket 并进⾏监听， 以等待客户端请求链接。如果该属性设置<br /> 为0， Tomcat将会随机选择⼀个可⽤的端⼝号给当前Connector 使⽤<br /> protocol：<br /> 当前Connector ⽀持的访问协议。 默认为 HTTP/1.1 ， 并采⽤⾃动切换机制选择⼀个基于 JAVA<br /> NIO 的链接器或者基于本地APR的链接器（根据本地是否含有Tomcat的本地库判定）<br /> connectionTimeOut:<br /> Connector 接收链接后的等待超时时间， 单位为 毫秒。 -1 表示不超时。<br /> redirectPort：<br /> 当前Connector 不⽀持SSL请求， 接收到了⼀个请求， 并且也符合security-constraint 约束，<br /> 需要SSL传输，Catalina⾃动将请求重定向到指定的端⼝。<br /> executor：<br /> 指定共享线程池的名称， 也可以通过maxThreads、minSpareThreads 等属性配置内部线程池。 
</p>

<p>
  URIEncoding:<br /> ⽤于指定编码URI的字符编码， Tomcat8.x版本默认的编码为 UTF-8 , Tomcat7.x版本默认为ISO-<br /> 8859-1<br /> --&gt;<br /> &lt;!--org.apache.coyote.http11.Http11NioProtocol ， ⾮阻塞式 Java NIO 链接器--&gt;<br /> &lt;Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000"<br /> redirectPort="8443" /&gt;<br /> &lt;Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /&gt; 
</p></pre>

可以使用共享线程池

<pre><p>
  &lt;Connector port="8080"<br />          protocol="HTTP/1.1"<br />          executor="commonThreadPool"<br />          maxThreads="1000"<br />          minSpareThreads="100"<br />          acceptCount="1000"<br />          maxConnections="1000"<br />          connectionTimeout="20000"<br />          compression="on"<br />          compressionMinSize="2048"<br />          disableUploadTimeout="true"<br />          redirectPort="8443"<br />          URIEncoding="UTF-8" /&gt; 
</p></pre>

**Engine标签：**  
Engine表示Servlet引擎。

<pre><p>
  &lt;!--<br /> name： ⽤于指定Engine 的名称， 默认为Catalina<br /> defaultHost：默认使⽤的虚拟主机名称， 当客户端请求指向的主机⽆效时， 将交由默认的虚拟主机处<br /> 理， 默认为localhost<br /> --&gt;<br /> &lt;Engine name="Catalina" defaultHost="localhost"&gt;<br /> ...<br /> &lt;/Engine&gt; 
</p></pre>

**Host标签：**  
Host标签用于配置一个虚拟主机。

<pre><p>
  &lt;Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true"&gt;<br /> ...<br /> &lt;/Host&gt; 
</p></pre>

**Context标签：**  
Context标签用于配置一个Web应用。

<pre><p>
  &lt;Host name="www.abc.com" appBase="webapps" unpackWARs="true"<br /> autoDeploy="true"&gt;<br /> &lt;!--<br /> docBase：Web应⽤⽬录或者War包的部署路径。可以是绝对路径，也可以是相对于 Host appBase的<br /> 相对路径。<br /> path：Web应⽤的Context 路径。如果我们Host名为localhost， 则该web应⽤访问的根路径为：<br /> http://localhost:8080/web_demo。<br /> --&gt;<br /> &lt;Context docBase="/Users/yingdian/web_demo" path="/web3"&gt;&lt;/Context&gt;<br /> &lt;Valve className="org.apache.catalina.valves.AccessLogValve"<br /> directory="logs"<br /> prefix="localhost_access_log" suffix=".txt"<br /> pattern="%h %l %u %t "%r" %s %b" /&gt;<br /> &lt;/Host&gt; 
</p></pre>

## <span class="ez-toc-section" id="3_Tomcat%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6%E5%89%96%E6%9E%90"></span>**3. Tomcat类加载机制剖析**<span class="ez-toc-section-end"></span> {#mce_3.wp-block-heading}

类加载过程：

  1. 自行编写的Java类（.java）；
  2. 编译为字节码文件（.class）；
  3. 将字节码文件加载到JVM内存中。

JVM启动时先将类加载器「ClassLoader（也是一个类）」读取到内存中，类加载器再读取其他类，这里指的其他类如各种jar包中的字节码文件，自行开发编译后的.class文件等。  
要说Tomcat的类加载机制，首先查看JVM的类加载机制，因为Tomcat类加载机制是在JVM类加载机制基础上做了一些变动。

### <span class="ez-toc-section" id="31_JVM%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6"></span>**3.1.&nbsp;JVM的类加载机制**<span class="ez-toc-section-end"></span> {#mce_11.wp-block-heading}

JVM的类加载机制中有一个重要的角色是类加载器（ClassLoader），类加载器有自己的体系，JVM内置了几种类加载器，包括：引导类加载器、扩展类加载器、系统类加载器，他们之间形成父子关系，通过Parent属性来定义这种关系，最终可以形成树形结构。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/JVM类加载机制-2.png" alt="" class="wp-image-669" /><figcaption>图11 类加载器父子关系</figcaption></figure>
</div>

  * 类加载器
      * 引导启动类加载器（BootstrapClassLoader）：C++编写，加载Java核心库java.*，比如rt.jar中的类，构造ExtClassLoader和AppClassLoader；
      * 扩展类加载器（ExtClassLoader）：Java编写，加载扩展库JAVA_HOME/lib/ext目录下的jar中的类，如classpath中的jre，javax.*或者java.ext.dir指定位置中的类；
      * 系统类加载器（SystemClassLoader/AppClassLoader）：默认的类加载器，搜索环境变量classpath中指定的路径；
      * 自定义类加载器：搜索编程人员指定的特殊目录。

**另外：用户可以自定义类加载器（Java编写，用户自定义的类加载器可加载指定路径的class文件）**

当JVM运行过程中，若用户自定义了类加载器去加载某些类时，会按照下面的步骤（父类委托机制）

  1. 用户自己的类加载器，把加载请求传给父加载器，父加载器再传给其父加载器，一直到加载器树的最顶层；
  2. 最顶层的类加载器首先会对其指定的位置进行加载，若加载不到就转交给子类；
  3. 如果一直到底层的类加载都没加载到，那么就会抛出异常：ClassNotFoundException

因此，从此过程中可以看出：若同样在classpath指定的路径中和自定义类加载器指定的的路径中存放相同的class，会优先加载classpath指定路径中的文件。

### <span class="ez-toc-section" id="32_%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%9C%BA%E5%88%B6"></span>**3.2.&nbsp;双亲委派机制**<span class="ez-toc-section-end"></span> {#mce_12.wp-block-heading}

#### <span class="ez-toc-section" id="321_%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%9C%BA%E5%88%B6%E7%AE%80%E8%BF%B0"></span>**3.2.1. 双亲委派机制简述**<span class="ez-toc-section-end"></span> {#mce_28.wp-block-heading}

当某个类加载器需要加载某个.class文件时，首先将任务委托给上级加载器。递归这个操作，若上级的类加载器没有加载，自身才会去加载此.class。

#### <span class="ez-toc-section" id="322_%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%9C%BA%E5%88%B6%E7%9A%84%E4%BD%9C%E7%94%A8"></span>**3.2.2. 双亲委派机制的作用**<span class="ez-toc-section-end"></span> {#mce_29.wp-block-heading}

  * 防止重复加载同一个.class。通过委托上级加载器，知道是否加载过某.class。若加载过，就不需要再加载，保证数据安全。
  * 保证核心.class不能被篡改。通过委托方式，即使篡改了核心.class，被篡改的.class也不会被加载，即使加载也不会是同一个.class对象。不同的加载器加载同一个.class也不是用一个.class对象，这样保证了class执行安全。（若不遵从双亲委派机制，由子类加载器先加载，那么作恶者可以写一些与java.lang包中基础类同名的类，然后再自定义一个子类加载器，这样应用的基础类就变成作恶者自定义的类了）

### <span class="ez-toc-section" id="33_Tomcat%E7%9A%84%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6"></span>**3.3.&nbsp;Tomcat的类加载机制**<span class="ez-toc-section-end"></span> {#mce_32.wp-block-heading}

Tomcat的类加载机制相对于JVM的类加载机制做了一些改变。没有严格的遵从双亲委派机制。<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/Tomcat类加载机制.png" alt="" class="wp-image-670" /> <figcaption>图12 Tomcat的类加载机制</figcaption></figure> 

  * 引导类加载器和扩展类加载器的作用不变；
  * 系统类加载器正常情况下加载的是classpath下的类，但是Tomcat的启动脚本并未使用该变量，而是加载Tomcat启动的类，比如bootstrap.jar，通常在catalina.bat或者catalina.sh中指定。位于CATALINA_HOME/bin下；
  * Common ClassLoader加载Tomcat使用以及应用通用的一些类，位于CATALINA_HOME/lib下，比如servlet-api.jar；
  * Catalina ClassLoader用于加载服务器内部可见类，这些类应用程序不能访问；
  * Shared ClassLoader用于加载应用程序共享类，这些类服务器不会依赖；
  * Webapp ClassLoader，每个应用程序都有一个独一无二的Webapp ClassLoader，他用来加载本应用程序/WEB-INF/classes和/WEB-INF/lib下的类

下面为Tomcat 8.5的类加载机制，它默认改变了严格的双亲委派机制：

  1. 首先从引导类加载器加载指定类；
  2. 如果未加载到，则Webapp ClassLoader从/WEB-INF/classes加载；
  3. 如果未加载到，则Webapp ClassLoader从/WEB-INF/lib/*.jar加载；
  4. 如果未加载到，则依此从System, Common, Shared加载（这一步尊从双亲委派机制）。

## <span class="ez-toc-section" id="4_Tomcat%E5%AF%B9HTTPS%E7%9A%84%E6%94%AF%E6%8C%81%E5%8F%8ATomcat%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%E7%AD%96%E7%95%A5"></span>**4. Tomcat对HTTPS的支持及Tomcat性能优化策略**<span class="ez-toc-section-end"></span> {#mce_4.wp-block-heading}

### <span class="ez-toc-section" id="41_Tomcat%E5%AF%B9HTTPS%E7%9A%84%E6%94%AF%E6%8C%81"></span>**4.1.&nbsp;Tomcat对HTTPS的支持**<span class="ez-toc-section-end"></span> {#mce_37.wp-block-heading}

使用JDK中的keytool工具生成免费的密钥库文件（证书）:

<pre>keytool -genkey -alias robin -keyalg RSA -keystore robin.keystore<br /></pre>

配置conf/server.xml

<pre>&lt;Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"<br /> maxThreads="150" schema="https" secure="true" SSLEnabled="true"&gt;<br />     &lt;SSLHostConfig&gt;<br />         &lt;Certificate<br /> certificateKeystoreFile="文件地址路径/robin.keystore" certificateKeystorePassword="robin123" type="RSA"<br /> /&gt;<br />     &lt;/SSLHostConfig&gt;<br /> &lt;/Connector&gt; </pre>

使用HTTPS协议访问8443端口（https://localhost:8443）

### <span class="ez-toc-section" id="42_Tomcat%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%E7%AD%96%E7%95%A5"></span>**4.2.&nbsp;Tomcat性能优化策略**<span class="ez-toc-section-end"></span> {#mce_50.wp-block-heading}

系统性能的衡量指标，主要是响应时间和吞吐量。Tomcat优化从两个方面进行：

  1. JVM虚拟机优化（优化内存模型）；
  2. Tomcat自身配置的优化（比如是否使用共享线程池？IO模型？）

#### <span class="ez-toc-section" id="421_%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%BF%90%E8%A1%8C%E4%BC%98%E5%8C%96%EF%BC%88%E5%8F%82%E6%95%B0%E8%B0%83%E6%95%B4%EF%BC%89"></span>**4.2.1. 虚拟机运行优化（参数调整）**<span class="ez-toc-section-end"></span> {#mce_29.wp-block-heading}

Java虚拟机的运行优化主要是内存分配和垃圾回收策略的优化：

  * 内存直接影响服务的运行效率和吞吐量；
  * 垃圾回收机制会不同程度地导致程序运行中断（不同的垃圾回收策略，垃圾回收次数和回收效率都是不同的）。

#### <span class="ez-toc-section" id="4211_JVM%E5%86%85%E5%AD%98%E7%9B%B8%E5%85%B3%E5%8F%82%E6%95%B0"></span>**4.2.1.1. JVM内存相关参数**<span class="ez-toc-section-end"></span> {#mce_83.wp-block-heading}<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/JVM参数.jpg" alt="" class="wp-image-673" /> </figure> 

JVM内存模型可以参考此文[**《JVM运行时数据区》**][1]

参数调整示例：

<pre>JAVA_OPTS="-server -Xms2048m -Xmx2048m -XX:MetaspaceSize=256m -<br /> XX:MaxMetaspaceSize=512m" </pre>

调整后可在JDK的bin目录下使用内存映射工具查看Tomcat中JVM内存配置，命令如下：

<pre>./jhsdb jmap --heap -pid 8481(Tomcat的PID)</pre>

#### <span class="ez-toc-section" id="4212_%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%EF%BC%88GC%EF%BC%89%E7%AD%96%E7%95%A5"></span>**4.2.1.2. 垃圾回收（GC）策略**<span class="ez-toc-section-end"></span> {#mce_90.wp-block-heading}

**垃圾回收性能指标**

  * 吞吐量：工作时间（排除GC时间）占总时间的百分比，工作时间并不仅是程序运行的时间，还包含内存分配时间；
  * 暂停时间：由垃圾回收导致的应用程序停止响应次数/时间

**垃圾收集器**（各类垃圾收集器的详细信息可参考书籍《深入理解JVM》）：

  * 串行收集器（Serial Collector）
  * 并行收集器（Parallel Collector）
  * 并发收集器（Concurrent Collector）
  * CMS收集器（Concurrent Mark Sweep Collector）
  * G1收集器（Garbage-First Garbage Collector）

**垃圾回收器参数**<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/垃圾回收器参数.jpg" alt="" class="wp-image-674" /> </figure> 

在Tomcat的bin/catalina.sh中，追加如下配置：

<pre>JAVA_OPTS="-XX:+UseConcMarkSweepGC" </pre>

#### <span class="ez-toc-section" id="422_Tomcat%E9%85%8D%E7%BD%AE%E8%B0%83%E4%BC%98"></span>**4.2.2. Tomcat配置调优**<span class="ez-toc-section-end"></span> {#mce_83.wp-block-heading}

**调整Tomcat线程池**<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/Tomcat线程池.jpg" alt="" class="wp-image-675" /> <figcaption>图13 Tomcat线程池</figcaption></figure> 

**调整Tomcat的连接器**<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/Tomcat的链接器.jpg" alt="" class="wp-image-676" /> </figure> 

**禁用AJP连接器**<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/禁用AJP链接器.jpg" alt="" class="wp-image-678" /> <figcaption>图14 禁用AJP连接器  
</figcaption></figure> 

**调整IO模式**

Tomcat8之前的版本默认使用BIO模式，对于每一个请求都创建一个线程来处理，不适合高并发，Tomcat8以后的默认使用NIO模式<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/11/IO.jpg" alt="" class="wp-image-679" /> <figcaption>图15 调整IO模型</figcaption></figure> 

当Tomcat并发性能由较高要求或出现瓶颈时，可以尝试APR模式，APR(Apache Portable Runtime)是从操作系统级别解决异步IO问题，使用时需要在操作系统上安装APR和Native，因为APR原理时使用JNI计数调用操作系统底层IO接口。

**动静分离**

可以使用Nginx + Tomcat结合的方案，Nginx负责静态资源访问，Tomcat负责jsp等动态资源访问处理。因为Tomcat不擅长处理静态资源。

 [1]: http://roliu.work/2020/06/13/jvm%e8%bf%90%e8%a1%8c%e6%97%b6%e6%95%b0%e6%8d%ae%e5%8c%ba/