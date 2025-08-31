---
title: 深信服社招一面
date: 2024-12-17T03:12:25+00:00
categories:
  - 面经
---

### <span class="ez-toc-section" id="%E8%83%8C%E6%99%AF"></span>背景<span class="ez-toc-section-end"></span> {.wp-block-heading}

  * 公司：深信服
  * 部门：云计算部
  * 第几面：一面
  * 方向：后端开发
  * 语言：Go
  * 投递渠道：Boss
  * 面试时间：2024.9.29 19:30
  * 面试方式：线上（showmebug.com）

### <span class="ez-toc-section" id="%E6%80%BB%E4%BD%93%E6%83%85%E5%86%B5"></span>总体情况<span class="ez-toc-section-end"></span> {.wp-block-heading}

面试时间大概持续1小时，前半小时聊项目，后半小时八股+算法。面试工具是showmebug.com浏览器在线面试工具。

面试官没开视频，语音听起来比较年轻，比迅雷的面试官更积极，思维更活跃，比较喜欢就着你的回答内容进行追问。

### <span class="ez-toc-section" id="%E9%A1%B9%E7%9B%AE"></span>项目<span class="ez-toc-section-end"></span> {.wp-block-heading}

  1. 说一下项目框架，请求从客户端开始的流程（我回答：客户端请求 -> CDN加速 -> 阿里SLB -> Nginx -> 网关层做鉴权和全链路日志 -> 微服务的实例节点）
  2. 你提到CDN加速，CDN加速只对静态资源有效，对你们短链接应该是没太大帮助的，这你知道吗？（其实这里面试官说的不大准确，腾讯的ECDN其实能对动态资源加速，但没跟面试官作过多纠缠）
  3. 还有你提到了SLB，SLB是什么有什么作用？SLB是做负载均衡的，有了解SLB调度策略吗？
  4. 你们微服务框架是什么
  5. 介绍一个你做的项目吧
  6. 开始问我写的第一个项目的难点&重点

### <span class="ez-toc-section" id="%E5%85%AB%E8%82%A1"></span>八股<span class="ez-toc-section-end"></span> {.wp-block-heading}

  1. go的切片数据结构是什么
  2. channel在什么情况下会panic
  3. goruntine的数据结构是什么？能简单讲一下吗？换一个问法吧，为什么goruntine能够使用channle去做安全的并发呢？
  4. 你们在go语言上遇到比较多的问题什么？（回答：内存泄漏，之前有个问题就是把map定义到了全局变量里面，然后有协程不断往里面加数据）
  5. 如果我给你这个全局变量的map不停的写，然后再不停的删，会泄漏吗？（回答：还是会泄漏，因为删除map里面的值，只是把bitmap里面的元素key-value置空，并不会回收内存空间）
  6. 如果遇到这种情况应该怎么解决呢？（回答：把map放到方法里面，就相当于变量存在栈里，这样方法结束的时候变量就被回收掉，不需要等待GC了）

### <span class="ez-toc-section" id="%E7%AE%97%E6%B3%95"></span>算法<span class="ez-toc-section-end"></span> {.wp-block-heading}

给一个字符串，找出“重复单词”的个数（不区分大小写）。“重复单词”的定义是连续两个单词是一样的，比如：“cat CAT dog”，1个重复；“cat dog cat”，没有重复。

题目做出来了， 但是测试用例有一些没过，我问面试官哪些用例没过，有可能漏了一下情况，我再看着改一下。面试官说不用了，你也差不多做出来了

### <span class="ez-toc-section" id="%E5%85%B6%E4%BB%96"></span>其他<span class="ez-toc-section-end"></span> {.wp-block-heading}

  1. 你觉得最自豪的一件事是什么？
  2. 你有什么要问我的吗？