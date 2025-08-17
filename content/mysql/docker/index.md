---
title: 在Docker CentOS7.x镜像中通过源码方式安装MySQL
date: 2021-05-19T06:40:14+00:00
categories:
  - MySQL
tags:
  - Docker
  - MySQL

---
最近需要将三台腾讯云服务器的应用全部迁移到一个Docker CentOS7.x的镜像中。其他应用迁移都比较顺利，唯安装MySQL遇到了一些问题。因为在容器中使用`systemctl/service`命令会出现下图的错误：<figure class="wp-block-image">

<img decoding="async" src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6a8d454f5414a4dbe2d694be2c700ed~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> </figure> 

虽然找到一些解决方案，但是需要在宿主机上操作，于是找了容器平台运营方，很遗憾他们回答：“无法解决”。这就无法使用我一贯安装MySQL的<a rel="noreferrer noopener" href="https://link.juejin.cn?target=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2Flinux-installation-yum-repo.html" target="_blank">yum源方式</a>，改成使用源码安装MySQL。以下记录我通过源码方式安装MySQL的过程。  


<!--more-->

<div id="ez-toc-container" class="ez-toc-v2_0_56_1 counter-hierarchy ez-toc-counter ez-toc-grey ez-toc-container-direction">
  <div class="ez-toc-title-container">
    <p class="ez-toc-title " >
      目录
    </p>
    
    <span class="ez-toc-title-toggle"><a href="#" class="ez-toc-pull-right ez-toc-btn ez-toc-btn-xs ez-toc-btn-default ez-toc-toggle" aria-label="Toggle Table of Content" role="button"><label for="item-6896f659e9d08" ><span class=""><span style="display:none;">Toggle</span><span class="ez-toc-icon-toggle-span"><svg style="fill: #999;color:#999" xmlns="http://www.w3.org/2000/svg" class="list-377408" width="20px" height="20px" viewBox="0 0 24 24" fill="none"><path d="M6 6H4v2h2V6zm14 0H8v2h12V6zM4 11h2v2H4v-2zm16 0H8v2h12v-2zM4 16h2v2H4v-2zm16 0H8v2h12v-2z" fill="currentColor"></path></svg><svg style="fill: #999;color:#999" class="arrow-unsorted-368013" xmlns="http://www.w3.org/2000/svg" width="10px" height="10px" viewBox="0 0 24 24" version="1.2" baseProfile="tiny"><path d="M18.2 9.3l-6.2-6.3-6.2 6.3c-.2.2-.3.4-.3.7s.1.5.3.7c.2.2.4.3.7.3h11c.3 0 .5-.1.7-.3.2-.2.3-.5.3-.7s-.1-.5-.3-.7zM5.8 14.7l6.2 6.3 6.2-6.3c.2-.2.3-.5.3-.7s-.1-.5-.3-.7c-.2-.2-.4-.3-.7-.3h-11c-.3 0-.5.1-.7.3-.2.2-.3.5-.3.7s.1.5.3.7z"/></svg></span></span></label><input aria-label="Toggle" aria-label="item-6896f659e9d08"  type="checkbox" id="item-6896f659e9d08" /></a></span>
  </div><nav>
  
  <ul class='ez-toc-list ez-toc-list-level-1 ' >
    <li class='ez-toc-page-1 ez-toc-heading-level-2'>
      <a class="ez-toc-link ez-toc-heading-1" href="http://43.133.160.241/?p=944/#1_%E4%B8%8B%E8%BD%BDMySQL" title="1. 下载MySQL">1. 下载MySQL</a>
    </li>
    <li class='ez-toc-page-1 ez-toc-heading-level-2'>
      <a class="ez-toc-link ez-toc-heading-2" href="http://43.133.160.241/?p=944/#2_%E7%8E%AF%E5%A2%83%E4%BE%9D%E8%B5%96" title="2. 环境依赖">2. 环境依赖</a>
    </li>
    <li class='ez-toc-page-1 ez-toc-heading-level-2'>
      <a class="ez-toc-link ez-toc-heading-3" href="http://43.133.160.241/?p=944/#3_%E5%AE%89%E8%A3%85MySQL" title="3. 安装MySQL">3. 安装MySQL</a><ul class='ez-toc-list-level-3'>
        <li class='ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-4" href="http://43.133.160.241/?p=944/#31_%E8%A7%A3%E5%8E%8B" title="3.1. 解压">3.1. 解压</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-5" href="http://43.133.160.241/?p=944/#32_%E5%88%9B%E5%BB%BA%E7%BC%96%E8%AF%91%E7%9B%AE%E5%BD%95" title="3.2. 创建编译目录">3.2. 创建编译目录</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-6" href="http://43.133.160.241/?p=944/#33_%E7%94%9F%E6%88%90%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83" title="3.3. 生成编译环境">3.3. 生成编译环境</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-7" href="http://43.133.160.241/?p=944/#34_%E5%AE%89%E8%A3%85%E7%BC%96%E8%AF%91" title="3.4. 安装编译">3.4. 安装编译</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-8" href="http://43.133.160.241/?p=944/#35_%E5%88%9B%E5%BB%BAmysql%E7%94%A8%E6%88%B7%E7%BB%84%E5%92%8C%E7%94%A8%E6%88%B7" title="3.5. 创建mysql用户组和用户">3.5. 创建mysql用户组和用户</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-9" href="http://43.133.160.241/?p=944/#36_%E8%AE%BE%E7%BD%AE%E6%9D%83%E9%99%90" title="3.6. 设置权限">3.6. 设置权限</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-10" href="http://43.133.160.241/?p=944/#37_%E5%88%9B%E5%BB%BAmycnf" title="3.7. 创建my.cnf">3.7. 创建my.cnf</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-11" href="http://43.133.160.241/?p=944/#38_%E5%88%9D%E5%A7%8B%E5%8C%96%E6%95%B0%E6%8D%AE%E5%BA%93" title="3.8. 初始化数据库">3.8. 初始化数据库</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-12" href="http://43.133.160.241/?p=944/#39_%E5%90%AF%E5%8A%A8%E6%95%B0%E6%8D%AE%E5%BA%93" title="3.9. 启动数据库">3.9. 启动数据库</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-13" href="http://43.133.160.241/?p=944/#310_%E8%AE%BE%E7%BD%AE%E7%99%BB%E5%BD%95MySQL" title="3.10. 设置登录MySQL">3.10. 设置登录MySQL</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-3'>
          <a class="ez-toc-link ez-toc-heading-14" href="http://43.133.160.241/?p=944/#311_%E5%85%8D%E5%AF%86%E7%99%BB%E5%BD%95" title="3.11. 免密登录">3.11. 免密登录</a>
        </li>
      </ul>
    </li>
  </ul></nav>
</div>

## <span class="ez-toc-section" id="1_%E4%B8%8B%E8%BD%BDMySQL"></span>1. 下载MySQL<span class="ez-toc-section-end"></span> {#1.wp-block-heading}

<pre class="wp-block-code"><code class="">wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-boost-5.7.34.tar.gz</code></pre>

版本说明（通过以上链接下载的MySQL的版本说明）：

  * 版本 **MySQL Community Server 5.7.34**
  * **Source Code**
  * **Generic Linux (Architecture Independent)**
  * **带boost版本**

## <span class="ez-toc-section" id="2_%E7%8E%AF%E5%A2%83%E4%BE%9D%E8%B5%96"></span>2. 环境依赖<span class="ez-toc-section-end"></span> {#2.wp-block-heading}

源码方式安装MySQL所需的环境依赖：

  * cmake
  * make
  * gcc
  * gcc-c++
  * bison
  * ncurses
  * ncurses-devel（devel是开发版本）

<pre class="wp-block-code"><code class="">yum -y install cmake make gcc gcc-c++ bison ncurses ncurses-devel</code></pre>

## <span class="ez-toc-section" id="3_%E5%AE%89%E8%A3%85MySQL"></span>3. 安装MySQL<span class="ez-toc-section-end"></span> {#3.wp-block-heading}

### <span class="ez-toc-section" id="31_%E8%A7%A3%E5%8E%8B"></span>3.1. 解压<span class="ez-toc-section-end"></span> {#3.1.wp-block-heading}

<pre class="wp-block-code"><code class="">tar -zxvf mysql-boost-5.7.34.tar.gz
cd mysql-5.7.34/</code></pre>

### <span class="ez-toc-section" id="32_%E5%88%9B%E5%BB%BA%E7%BC%96%E8%AF%91%E7%9B%AE%E5%BD%95"></span>3.2. 创建编译目录<span class="ez-toc-section-end"></span> {#3.2.wp-block-heading}

<pre class="wp-block-code"><code class="">mkdir configure
cd configure</code></pre>

### <span class="ez-toc-section" id="33_%E7%94%9F%E6%88%90%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83"></span>3.3. 生成编译环境<span class="ez-toc-section-end"></span> {#3.3.wp-block-heading}

<pre class="wp-block-code"><code class="">cmake .. -DWITH_BOOST=../boost/boost_1_59_0/ -DWITH_EMBEDDED_SERVER=OFF</code></pre>

### <span class="ez-toc-section" id="34_%E5%AE%89%E8%A3%85%E7%BC%96%E8%AF%91"></span>3.4. 安装编译<span class="ez-toc-section-end"></span> {#3.4.wp-block-heading}

<pre class="wp-block-code"><code class="">make && make install</code></pre>

### <span class="ez-toc-section" id="35_%E5%88%9B%E5%BB%BAmysql%E7%94%A8%E6%88%B7%E7%BB%84%E5%92%8C%E7%94%A8%E6%88%B7"></span>3.5. 创建mysql用户组和用户<span class="ez-toc-section-end"></span> {#3.5.wp-block-heading}

<pre class="wp-block-code"><code class="">groupadd mysql
useradd -g mysql mysql</code></pre>

### <span class="ez-toc-section" id="36_%E8%AE%BE%E7%BD%AE%E6%9D%83%E9%99%90"></span>3.6. 设置权限<span class="ez-toc-section-end"></span> {#3.6.wp-block-heading}

MySQL通过`3.4. 安装编译`步骤完成后所在的目录为 /usr/local/mysql

<pre class="wp-block-code"><code class="">chown -R mysql:mysql /usr/local/mysql</code></pre>

### <span class="ez-toc-section" id="37_%E5%88%9B%E5%BB%BAmycnf"></span>3.7. 创建my.cnf<span class="ez-toc-section-end"></span> {#3.7.wp-block-heading}

<pre class="wp-block-code"><code class="">vim /etc/my.cnf</code></pre>

my.cnf里面的内容为如下：

<pre class="wp-block-code"><code class="">For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
datadir=/usr/local/mysql/data
socket=/usr/local/mysql/mysql.sock

character-set-client-handshake=FALSE
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

explicit_defaults_for_timestamp=true</code></pre>

以上内容也可自行调整

### <span class="ez-toc-section" id="38_%E5%88%9D%E5%A7%8B%E5%8C%96%E6%95%B0%E6%8D%AE%E5%BA%93"></span>3.8. 初始化数据库<span class="ez-toc-section-end"></span> {#3.8.wp-block-heading}

<pre class="wp-block-code"><code class="">cd /usr/local/mysql/bin
./mysqld --initialize --user=mysql</code></pre>

我这边没有显示初始化密码，需要成功启动数据库跳过密码进入MySQL再修改密码。

### <span class="ez-toc-section" id="39_%E5%90%AF%E5%8A%A8%E6%95%B0%E6%8D%AE%E5%BA%93"></span>3.9. 启动数据库<span class="ez-toc-section-end"></span> {#3.9.wp-block-heading}

<pre class="wp-block-code"><code class="">cd /usr/local/mysql/support-files
./mysql.server start</code></pre>

由于文章开始说过我这边的容器中无法使用`systemctl/service`命令，所以我没有把MySQL加入系统服务，有需要的读者可自行加入。

### <span class="ez-toc-section" id="310_%E8%AE%BE%E7%BD%AE%E7%99%BB%E5%BD%95MySQL"></span>3.10. 设置登录MySQL<span class="ez-toc-section-end"></span> {#3.10.wp-block-heading}

通过一下命令创建软连接，可直接在命令行使用`mysql`命令。

<pre class="wp-block-code"><code class="">ln -s /usr/local/mysql/bin/mysql /usr/local/bin</code></pre>

创建软链接到/tmp/mysql.sock

<pre class="wp-block-code"><code class="">ln -s /usr/local/mysql/mysql.sock /tmp/mysql.sock</code></pre>

### <span class="ez-toc-section" id="311_%E5%85%8D%E5%AF%86%E7%99%BB%E5%BD%95"></span>3.11. 免密登录<span class="ez-toc-section-end"></span> {#3.11.wp-block-heading}

修改/etc/my.cnf，添加`skip-grant-tables`

重启MySQL

<pre class="wp-block-code"><code class="">cd /usr/local/mysql/support-files
./mysql.server restart</code></pre>

免密进入MySQL命令行

<pre class="wp-block-code"><code class="">mysql</code></pre>

修改登录密码

<pre class="wp-block-code"><code class="">use mysql;
update user set authentication_string=password('PASSWORD') where user='root';
flush privileges;</code></pre>

最后，退出MySQL命令行，将/etc/my.cnf中的`skip-grant-tables`删除或注释掉，再次重启MySQL就可以使用新修改的密码登录。

PS：以上是我在掘金写的文章，所以图片带水印。