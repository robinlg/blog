---
title: Setting tomcat8 as system services on Centos7
date: 2020-02-12T03:11:58+00:00
categories:
  - Linux
summary: 详细介绍如何在CentOS7上将Tomcat8配置为系统服务，包括解压安装、配置JAVA_HOME、创建systemd服务文件和启动管理。
---
The following methods are effective for me. Here&#8217;s how I did it.  
*** I&#8217;m operating in /home/robin**

### <span class="ez-toc-section" id="1_Extracting_%E2%80%9Ctomcattargz%E2%80%9D_to_homerobin"></span>**1.** Extracting &#8220;tomcat*.tar.gz&#8221; to /home/robin  
  
<span class="ez-toc-section-end"></span> {.wp-block-heading}

### <span class="ez-toc-section" id="2_Typeing_command_vim_homerobintomcatbincatalinash"></span>**2.** Typeing command: vim /home/robin/tomcat*/bin/catalina.sh<span class="ez-toc-section-end"></span> {.wp-block-heading}

**JAVA\_HOME=/opt/jdk1.8.0\_202  
JRE\_HOME=$JAVA\_HOME/jre**  
Adding above in from of &#8220;# OS specific support.&#8221;(  
Note the Java installation directory )  
<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/02/catalina.png" alt="" class="wp-image-515" /> <figcaption> catalina.sh </figcaption></figure> 

### <span class="ez-toc-section" id="3_Modifiying_%E2%80%9Cusrlibsystemdsystemtomcatservice%E2%80%9D_You_can_change_it_to_any_name_you_like_eg_mytomcatservice_mycommandservice_and_so_on_Typeing"></span>**3**. Modifiying &#8220;/usr/lib/systemd/system/**tomcat**.service&#8221; (You can change it to any name you like. e.g. **mytomcat**.service, **mycommand**.service and so on. )  
  
Typeing: <span class="ez-toc-section-end"></span> {.wp-block-heading}

<p class="has-very-light-gray-background-color has-background">
  [Unit]<br />Description=Tomcat<br />After=syslog.target network.target remote-fs.target nss-lookup.target <br /><br />[Service]<br />Type=oneshot<br />ExecStart=/usr/local/tomcat/bin/startup.sh<br />ExecStop=/usr/local/tomcat/bin/shutdown.sh<br />ExecReload=/bin/kill -s HUP $MAINPID<br />RemainAfterExit=yes <br /><br />[Install]<br />WantedBy=multi-user.target
</p>

### <span class="ez-toc-section" id="4_Allow_start-up"></span>**4**. Allow start-up<span class="ez-toc-section-end"></span> {.wp-block-heading}

systemctl enable tomcat 

### <span class="ez-toc-section" id="5_Commond_of_Tomcat"></span>**5**. Commond of Tomcat<span class="ez-toc-section-end"></span> {.wp-block-heading}

<p class="has-text-align-left">
  systemctl start tomcat.service <br /> systemctl status tomcat.service <br /> systemctl restart tomcat.service <br /> systemctl stop tomcat.service
</p>

The original link： https://www.cnblogs.com/wycfm/p/9595550.html