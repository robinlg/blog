---
title: MySQL简介
date: 2021-04-30T00:00:00+00:00
categories:
  - MySQL
tags:
  - MySQL
---
最近在学习MySQL的知识，为方便自己查阅笔记，接下来会将一系列MySQL相关知识记录于此博客。想了解MySQL知识的读者，可以持续关注本系列的文章。本文会从下面三部分展开：

  1. **MySQL起源**
  2. **MySQL分支**
  3. **MySQL应用架构演变**

## **MySQL起源** {.wp-block-heading}

MySQL 是最流行的关系型数据库软件之一，由于其体积小、速度快、开源免费、简单易用、维护成本低等，在集群架构中易于扩展、高可用，因此深受开发者和企业的欢迎。

在[DB-Engines][1]可以查看各数据库的排名<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2021/04/截屏2021-04-18-上午11.43.35-1024x362.png" alt="" class="wp-image-830" /> </figure> 

从排名可以看出Oracle和MySQL是世界市场占比最高的两种数据库。在IT行业早期，很多企业都将系统IOE化（IBM服务器，Oracle数据库，EMC储存设备）。一些大大企业仍在使用IOE化的架构，如银行、电信、石油、证券这些非互联网企业。MySQL往往是大多数互联网企业的首选。<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2021/04/MySQL海量数据存储与优化（上）-802x1024.jpg" alt="" class="wp-image-833" /> <figcaption>MySQL发展历程</figcaption></figure> 

## **MySQL主流分支** {.wp-block-heading}

 [1]: https://db-engines.com/en/ranking