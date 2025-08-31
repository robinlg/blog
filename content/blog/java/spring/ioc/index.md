---
title: Spring IoC基础与源码剖析
date: 2020-09-05T09:38:18+00:00
categories:
  - Java
---
<figure class="wp-block-image"><img decoding="async" src="http://roliu.work/wp-content/uploads/2020/09/IoC基础-1024x449.png" alt="" class="wp-image-624" /></figure> 

Bean的概念在Spring框架出现之前就已经被提出了，笔者通过此篇文章剖析了Spring Bean在Spring IoC（控制反转）中「如何被生成」以及「如何被管理」。  


<p class="has-luminous-vivid-orange-color has-text-color has-small-font-size">
  <strong>注意：在本文中，bean和beanFactory开头大小写遵照Java语法中类与对象的定义。</strong>
</p>

从上图中可以看出bean可以通过三种方式定义，无论是由XML文件还是注解定义，作用都是「定义需要实例化的类」和「描述类之间的依赖」。

beanFactory通过反射技术来实例化对象并维护对象之间的关系，BeanFactory在Spring框架中是顶层接口（会在后面源码分析的部分介绍），实际上bean是被其子接口ApplicationContext的实现类来管理，实现类均表示在上图中。

* 好处：提高培养代码架构思维，深入理解框架
    * 原则
        * 定焦原则：抓主线
        * 宏观原则：站在上帝视角，关注源码结构和业务流程（淡化具体某行代码的执行细节）
    * 读源码的方法和技巧
        * 断点（观察调用栈）
        * 反调（Find Usages）
        * 经验（Spring框架中doXXX是具体做处理的地方）
    * Spring源码构建
        * 下载源码（Github）
        * 安装gradle5.6.3或以上
        * 编译工程（顺序：core -> oxm -> context -> beans -> aspects -> aop）
            * 工程 -> tasks -> other -> compileTestJava

<p class="has-vivid-cyan-blue-color has-text-color has-small-font-size">
  PS：源码剖析部分在今后将会补充至本文下方。
</p>

