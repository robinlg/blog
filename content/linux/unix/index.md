---
title: UNIX环境高级编程——UNIX标准及实现
date: 2023-09-17T08:35:36+00:00
categories:
  - Linux
tags:
  - UNIX环境高级编程

---

## <span class="ez-toc-section" id="UNIX%E6%A0%87%E5%87%86"></span>UNIX标准<span class="ez-toc-section-end"></span> {.wp-block-heading}

### <span class="ez-toc-section" id="ISO_C"></span>ISO C<span class="ez-toc-section-end"></span> {.wp-block-heading}

1989年下半年，C程序设计语言的ANSI标准X3.159-1989得到批准。此标准被也采纳为国际标准ISO/IEC 9899:1990。ANSI是美国国家标准学会（American National Standards Institute）的缩写，它是国际标准化组织（International Organization for Standardization，ISO）中代表美国的成员。IEC是国际电子技术委员会（International Electrotechnical Commission）的缩写。

### <span class="ez-toc-section" id="IEEE_POSIX"></span>IEEE POSIX<span class="ez-toc-section-end"></span> {.wp-block-heading}

POSIX是一个最初由IEEE（Institute of Electrical and Electronics Engineers，电气和电子工程师学会）制订的标准族。POSIX指的是可移植操作系统接口（Portable Operating System Interface）。它原来指的只是IEEE标准1003.1-1988（操作系统接口），后来则扩展成包括很多标记为1003的标准及标准草案，如shell和实用程序（1003.2）。

### <span class="ez-toc-section" id="Single_UNIX_Specification"></span>Single UNIX Specification<span class="ez-toc-section-end"></span> {.wp-block-heading}

Single UNIX Specification（SUS，单一UNIX规范）是POSIX.1标准的一个超集，它定义了一些附加接口扩展了POSIX.1规范提供的功能。POSIX.1相当于Single UNIX Specification中的基本规范部分。

## <span class="ez-toc-section" id="UNIX%E7%B3%BB%E7%BB%9F%E5%AE%9E%E7%8E%B0"></span>UNIX系统实现<span class="ez-toc-section-end"></span> {.wp-block-heading}

上一节说明了3个由各自独立的组织所制定的标准：ISO C、IEEE POSIX以及Single UNIX Specification。但是，标准只是接口的规范。这些标准是如何与现实世界相关连的呢？这些标准由厂商采用，然后转变成具体实现。

### <span class="ez-toc-section" id="SVR4"></span>SVR4<span class="ez-toc-section-end"></span> {.wp-block-heading}

SVR4（UNIX System V Release 4）是AT&T的UNIX系统实验室（UNIX System Laboratories， USL，其前身是AT&T的UNIX Software Operation）的产品，它将下列系统的功能合并到了一个一致的操作系统中：AT&T UNIX系统V 3.2版（SVR3.2）、Sun Microsystems公司的SunOS操作系统、加州大学伯克利分校的4.3BSD以及微软的Xenix系统（Xenix是在V7的基础上开发的，后来又采纳了很多系统V的功能）。其源代码于1989年后期发布，在1990年开始向终端用户提供。SVR4符合POSIX 1003.1标准和X/Open XPG3标准。

### <span class="ez-toc-section" id="44BSD"></span>4.4BSD<span class="ez-toc-section-end"></span> {.wp-block-heading}

BSD（Berkeley Software Distribution）是由加州大学伯克利分校的计算机系统研究组（CSRG）研究开发和分发的。4.2BSD于1983年问世，4.3BSD则于1986年发布。这两个版本都在VAX小型机上运行。它们的下一个版本4.3BSD Tahoe于1988年发布，在一台称为Tahoe的小型机上运行（Leffler等[1989]说明了4.3BSD Tahoe版）。其后又有1990年的4.3BSD Reno版，它支持很多POSIX.1的功能。

### <span class="ez-toc-section" id="FreeBSD"></span>FreeBSD<span class="ez-toc-section-end"></span> {.wp-block-heading}

FreeBSD基于4.4BSD-Lite 操作系统。在加州大学伯克利分校的 CSRG决定终止其在UNIX操作系统的BSD版本的研发工作，而且386BSD项目被忽视很长时间之后，为了继续坚持BSD系列，形成了FreeBSD项目。

### <span class="ez-toc-section" id="Linux"></span>Linux<span class="ez-toc-section-end"></span> {.wp-block-heading}

Linux是一种提供类似于UNIX的丰富编程环境的操作系统，在GNU公用许可证的指导下， Linux是免费使用的。Linux的普及是计算机产业中的一道亮丽风景线。Linux经常是支持较新硬件的第一个操作系统，这一点使其引人注目。  
Linux是由Linus Torvalds在1991年为替代MINIX而研发的。一位当时名不见经传人物的努力掀起了澎湃巨浪，吸引了遍布全世界的很多软件开发者，在使用和不断增强Linux方面自愿贡献出了他们大量的时间。

### <span class="ez-toc-section" id="Mac_OS_X"></span>Mac OS X<span class="ez-toc-section-end"></span> {.wp-block-heading}

与其以前的版本相比，Mac OS X使用了完全不同的技术。其核心操作系统称为“Darwin”，它基于Mach内核（Accetta等[1986]）、FreeBSD操作系统以及具有面向对象框架的驱动和其他内核扩展的结合。Mac OS X 10.5的Intel部分已经被验证为是一个UNIX系统。

### <span class="ez-toc-section" id="Solaris"></span>Solaris<span class="ez-toc-section-end"></span> {.wp-block-heading}

Solaris是由Sun Microsystems（现为Oracle）开发的UNIX系统版本。它基于SVR4，在超过15 年的时间里，Sun Microsystems 的工程师对其功能不断增强。它是唯一在商业上取得成功的SVR4后裔，并被正式验证为UNIX系统。

还有一些已验证的UNIX系统就不一一列举了。

## <span class="ez-toc-section" id="%E6%A0%87%E5%87%86%E4%B9%8B%E9%97%B4%E7%9A%84%E5%86%B2%E7%AA%81"></span>标准之间的冲突<span class="ez-toc-section-end"></span> {.wp-block-heading}

就整体而言，这些不同的标准之间配合得相当好。因为 SUS 基本说明和 POSIX.1 是同一个东西，所以我们不对它们进行特别的说明，我们主要关注ISO C标准和POSIX.1之间的差别。它们之间的冲突并非有意，但如果出现冲突，POSIX.1服从ISO C标准。然而它们之间还是存在着一些差别的。

另一个可能产生冲突的地方是：在ISO C标准说明函数时，可能没有像POSIX.1那样严。在POSIX环境下，有些函数可能要求有一个与C环境下不同的实现，因为POSIX环境中有多个进程，而ISO C环境则很少考虑宿主操作系统。