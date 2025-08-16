---
title: Shell的内建命令
date: 2019-12-30T08:07:18+00:00
categories:
  - Linux
tags:
  - 内建命令
---
上周笔者在使用Linux服务器时频繁遇到以下问题，大概能猜测到问题的原因，也知道如何解决。可一直无法找到相对官方和详细的解释。可最近书上看到了详细的说明，便记录一下。

### <span class="ez-toc-section" id="%E9%97%AE%E9%A2%98%EF%BC%9A"></span>**问题：**<span class="ez-toc-section-end"></span> {.wp-block-heading}

在Windows上使用Putty连接Linux云服务器，输入命令

###### <span class="ez-toc-section" id="_export_PATHPATH_optgradle-52bin"></span> $ export PATH=$PATH:/opt/gradle-5.2/bin <span class="ez-toc-section-end"></span> {.wp-block-heading}

按下回车后gradle命令能正常使用，但将Putty关闭后再打开，服务器会提示找不到gradle命令。再次输入以上命令仅能暂时解决问题，除非修改~/.bashrc（针对当前登录用户）或者/etc/profile（针对所有用户）。

### <span class="ez-toc-section" id="%E5%8E%9F%E5%9B%A0%EF%BC%9A"></span>**原因：**<span class="ez-toc-section-end"></span> {.wp-block-heading}

export属于shell的内建命令，像cd和exit命令都内建与bash shell。可以用过type命令来了解某个命令是否是内建的。

###### <span class="ez-toc-section" id="_type_cd_cd_is_a_shell_builtin"></span>$ **type cd**  
cd is a shell builtin<span class="ez-toc-section-end"></span> {.wp-block-heading}

###### <span class="ez-toc-section" id="_type_export_export_is_a_shell_builtin"></span> $ **type export**  
export is a shell builtin <span class="ez-toc-section-end"></span> {.wp-block-heading}

使用内建命令设置的环境变量仅在当前的shell进程中才有效。

放在笔者的例子上，简单点来说，使用内建命令所产生的作用范围仅仅在当前的Putty的图形输入框中。当关掉当前Putty图形输入框，内建命令所产生的“效果”就会消失。

### <span class="ez-toc-section" id="%E5%86%8D%E4%B8%BE%E4%B8%80%E4%B8%AA%E4%BE%8B%E5%AD%90%EF%BC%9A"></span>**再举一个例子：**  
<span class="ez-toc-section-end"></span> {.wp-block-heading}

可以使用alias命令创建属于自己的别名

###### <span class="ez-toc-section" id="_alias_lils_-li"></span>$ **alias li=&#8217;ls -li&#8217;**<span class="ez-toc-section-end"></span> {.wp-block-heading}

在定义好别名之后，你随时都可以在shell中使用它，就算在shell脚本中也没问题。要注意，因为命令别名属于内部命令，一个别名仅在它所被定义的shell进程中才有效。

参考资料： 

  * [1]Richard Blum & Christine Bresnahana. Linux Command Line and Shell Scripting Bible, 3E.(5.3.2内建命令)