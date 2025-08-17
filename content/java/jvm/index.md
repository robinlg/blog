---
title: JVM运行时数据区
date: 2020-06-13T07:58:23+00:00
categories:
  - Java
tags:
  - JVM

---
根据《Java虚拟机规范》的规定，Java虚拟机所管理的内存将会包括以下几个运行时数据区域，如图1所示。<figure class="wp-block-image">


### <span class="ez-toc-section" id="%E7%A8%8B%E5%BA%8F%E8%AE%A1%E6%95%B0%E5%99%A8"></span>**程序计数器**<span class="ez-toc-section-end"></span> {.wp-block-heading}

程序计数器（Program Counter Register）是一块较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器。在Java虚拟机的概念模型里，字节码解释器工作时就是通过改变这个计数器对的值来选取下一条需要执行的字节码指令，它是程序控制流的指示器，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器完成。

由于JVM的多线程是通过线程轮流切换、分配处理器执行时间的方式来实现的。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。

### <span class="ez-toc-section" id="Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%A0%88"></span>**Java虚拟机栈**<span class="ez-toc-section-end"></span> {.wp-block-heading}

与程序计数器一样，Java虚拟机栈（Java Virtual Machine Stack）也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是Java方法执行的线程内存模型：每个方法被执行的时候，Java虚拟机都会同步创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法被调用直至执行完毕的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

### <span class="ez-toc-section" id="%E6%9C%AC%E5%9C%B0%E6%96%B9%E6%B3%95%E6%A0%88"></span>**本地方法栈**<span class="ez-toc-section-end"></span> {.wp-block-heading}

本地方法栈（Native Method Stacks）与虚拟机所发挥的作用是非常相似的，其区别只是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的本地（Native）方法服务。

### <span class="ez-toc-section" id="Java%E5%A0%86"></span>**Java堆**<span class="ez-toc-section-end"></span> {.wp-block-heading}

对于Java应用程序来说，Java堆（Java Heap）是虚拟机所管理的内存中最大的一块。Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，Java世界里“几乎”所有的对象实例都在这里分配内存。书的作者在这里使用“几乎”是从实现角度来看，随着Java语言的发展，现在已经能看到些许迹象表明日后可能出现值类型的支持。

如果从分配内存的角度看，所有线程共享的Java堆中可以划分出多个线程私有的分配缓存区（Thread Local Allocation Buffer, TLAB），以提升对象分配时的效率。不过无论从什么角度，无论如何划分，都不会改变Java堆中内存储内容的共性，无论是哪个区域，存储的都只能是对象的实例，将Java堆细分的目的只是为了更好地回收内存，或者更快地分配内存。

### <span class="ez-toc-section" id="%E6%96%B9%E6%B3%95%E5%8C%BA"></span>**方法区**<span class="ez-toc-section-end"></span> {.wp-block-heading}

方法区（Method Area）与Java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。虽然《Java虚拟机规范》中把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫作“非堆”（Non-Heap），目的是与Java堆区分开来。

在JDK8以前，许多Java程序员更愿意把方法区称为“永久代”（Permanent Generation），或将两者混为一谈。其实仅仅是当时HotSpot虚拟机设计团队选择把收集器的分代设计扩展至方法区，或者说使用永久代来实现方法区而已。其实当年使用永久代来实现方法区的决定并不是一个好主意，这种设计导致Java应用更容易遇到内存移除的问题。

考虑到HotSpot未来的发展，在JDK6的时候HotSpot开发团队就有放弃永久代，逐步改为采用本地内存（Native Memory）来实现方法区的计划，到了JDK7的HotSpot，已经把原本放在永久代的字符串常量池、静态变量等移出，而到了JDK8，完全放弃了永久代的概念，改为与JRockit、J9一样在本地内存中实现的元空间（Meta-space）来代替，把JDK7中永久代还剩余的内容（主要是类型信息）全部移到元空间中。

### <span class="ez-toc-section" id="%E8%BF%90%E8%A1%8C%E6%97%B6%E5%B8%B8%E9%87%8F%E6%B1%A0"></span>**运行时常量池**<span class="ez-toc-section-end"></span> {.wp-block-heading}

运行时常量池（Runtime Constant Pool）是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池表（Constant Pool Table），用于存放编译期生成的各种字面量与符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。

### <span class="ez-toc-section" id="%E7%9B%B4%E6%8E%A5%E5%86%85%E5%AD%98"></span>**直接内存**<span class="ez-toc-section-end"></span> {.wp-block-heading}

直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是《Java虚拟机规范》中定义的内存。

在JDK1.4中新加入了NIO（New Input/Output）类，引入了一种基于通道（Channel）与缓存区（Buffer）的I/O方式，它可以使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆里面的DirectByteBuffer对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java堆和Native堆中来回复制数据。



<p class="has-vivid-cyan-blue-color has-text-color">
  文章参考《深入理解Java虚拟机》第3版.周志明
</p>