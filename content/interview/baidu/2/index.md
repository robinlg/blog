---
title: 百度社招二面
date: 2025-01-10T08:46:30+00:00
categories:
  - 面经
tags:
  - 面经

---

### <span class="ez-toc-section" id="%E8%83%8C%E6%99%AF"></span>背景<span class="ez-toc-section-end"></span> {.wp-block-heading}

  * 公司：百度
  * 部门：地图
  * 第几面：二面
  * 方向：后端开发
  * 语言：Go
  * 投递渠道：Boss
  * 面试时间：2024.10.14 20:00
  * 面试方式：线上（如流）

### <span class="ez-toc-section" id="%E6%80%BB%E4%BD%93%E4%BB%8B%E7%BB%8D"></span>总体介绍<span class="ez-toc-section-end"></span> {.wp-block-heading}

面试大概持续了1小时15分钟，00:00-00:25聊第一间公司的项目，00:25-00:30聊第二间公司的项目，00:30-00:45问八股，00:45-01:05写算法，01:05-01:15问其它。

百度一、二面给我的感觉是纯八股比较少，挺多问题是贴近实际开发的。

### <span class="ez-toc-section" id="%E9%A1%B9%E7%9B%AE"></span>项目<span class="ez-toc-section-end"></span> {.wp-block-heading}

#### <span class="ez-toc-section" id="%E4%BB%8B%E7%BB%8D%E7%AC%AC%E4%B8%80%E9%97%B4%E5%85%AC%E5%8F%B8%E7%9A%84%E9%A1%B9%E7%9B%AE"></span>介绍第一间公司的项目<span class="ez-toc-section-end"></span> {.wp-block-heading}

追问简历第一个项目的业务流程、难点、亮点、实现细节

#### <span class="ez-toc-section" id="%E7%AE%80%E5%8D%95%E4%BB%8B%E7%BB%8D%E7%AC%AC%E4%BA%8C%E9%97%B4%E5%85%AC%E5%8F%B8%E7%9A%84%E9%A1%B9%E7%9B%AE"></span>简单介绍第二间公司的项目<span class="ez-toc-section-end"></span> {.wp-block-heading}

追问项目业务、做了什么

### <span class="ez-toc-section" id="%E5%85%AB%E8%82%A1"></span>八股<span class="ez-toc-section-end"></span> {.wp-block-heading}

  1. HTTP协议主要有哪些字段？报文长度是用的哪个字段？
  2. DNS是用作域名解析，里面又个CName的作用是什么？
  3. 如果在浏览器输入百度的域名，发现页面打不开，排查思路是怎么样的？
  4. 我看你简历MySQL是比较熟悉的，关于数据库有时遇到这样一种场景：单表记录特别多，现在有没有什么其他数据库可以不限制单表记录数量，然后也可以和MySQL基本上是兼容的？（我答的TiDB）
  5. MySQL在日常工作中有遇到过什么解决不了的问题吗？（我答的索引问题，引申出了MySQL的优化器是基于成本分析的，然后用analyse打印出不同SQL的走索引的成本，也应该和我预想的一样。但实际和预想的不一样，已经提了阿里云工单，暂未回复）
  6. 如果想在数据库里面存视频或者文本，用MySQL会有问题吗？（我答用OSS存这些对象，MySQL只存链接）
  7. 那链接的时效性怎么解决？（MySQL只存相对路径，向外暴露的链接用OSS的SDK产生，也可以指定链接的时效性）
  8. 如果网站被攻击了，我想看攻击是来自哪个IP，怎么看？Linux命令是什么？
  9. 如果定位到IP，要把他加到防火墙，怎么加？Linux命令是什么？

### <span class="ez-toc-section" id="%E7%AE%97%E6%B3%95"></span>算法<span class="ez-toc-section-end"></span> {.wp-block-heading}

给你一个有序链表，需要把链表里重复元素删掉，删除后不能乱序

### <span class="ez-toc-section" id="%E5%85%B6%E5%AE%83"></span>其它<span class="ez-toc-section-end"></span> {.wp-block-heading}

  * 你有什么想问我的吗？ 
      * 您觉得我这次面试表现怎么样？（通过面试官回答，我觉得差不多稳了）
      * 如果我加入部门，这个职位的工作是什么？（面试官很认真的介绍了岗位职责，这么认真的介绍一般都是稳了。最后还说了句“今天就先这样，我会把面试结论反馈上去，到时候保持电话畅通”，加上这句就稳中加稳了）