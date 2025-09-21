---
title: 普渡科技一面
date: 2025-01-09T02:28:26+00:00
categories:
  - 面经
summary: 普渡科技云服务部门Go后端开发面试，重点考察Go语言GC机制、channel原理、HTTP协议、MySQL索引、Redis缓存等核心技术。
---

### <span class="ez-toc-section" id="%E8%83%8C%E6%99%AF"></span>背景<span class="ez-toc-section-end"></span> {.wp-block-heading}

  * 公司：普渡科技
  * 部门：云服务
  * 第几面：一面
  * 方向：后端开发
  * 语言：Go
  * 投递渠道：Boss
  * 面试时间：2024.10.15 19:00
  * 面试方式：线上（腾讯会议）

### <span class="ez-toc-section" id="%E6%80%BB%E4%BD%93%E6%83%85%E5%86%B5"></span>总体情况<span class="ez-toc-section-end"></span> {.wp-block-heading}

面试持续1小时左右，因为面试的部门主要是做k8s相关，所以没有怎么考察项目。主要是问了一些代码规范、开发流程、八股、算法。

### <span class="ez-toc-section" id="%E5%85%AB%E8%82%A1"></span>八股<span class="ez-toc-section-end"></span> {.wp-block-heading}

  1. channel的底层结构说一下？channel怎么保证读写并发安全？读写队列的流程再详细说下
  2. GC演进的流程说一下
  3. GC写屏障和混合写屏障有什么不同？怎么理解写屏障？
  4. GC的大概流程说一下（开启写屏障stop the world -> 三色标记 -> 关闭写屏障 -> 内存的清理 -> 内存的释放）
  5. RESTful是怎么理解的
  6. 你们怎么做单元测试？对覆盖率有没有要求？
  7. 说一下HTTP三次握手的流程
  8. 为什么要三次握手？二次握手不行吗？
  9. 说一下HTTP四次挥手的流程
 10. 你在四次挥手中提到MSL，它的作用是什么？
 11. 在日常的网络请求中，有一些网络攻击，比如像DDOS，怎么去做一些预防、防范措施呢？
 12. MySQL索引的底层原理说一下
 13. 什么情况下要回表？
 14. 为什么用B+树，不用B树
 15. Redis Zset的底层实现说一下
 16. 热点数据做缓存的时候，一般怎么处理？（实际问的就是缓存击穿处理）
 17. 如果Redis中有上亿个以统一某个前缀开头的key，我要查其中几个，怎么样查会更好？（我回答用scan以及它的用法）
 18. 如果让你去设计一个面对ToC，且订单量比较大的web架构，怎么做？
 19. 你提到了网关，如果网关挂了你怎样让他快速起来？除了加一些限流措施，从运维角度有办法吗？现在你们公司没用k8s将它自动拉起吗？
 20. 微服务的优点和痛点说一下
 21. MySQL的性能调优方法说一下
 22. go的性能调优说一下
 23. 你们平常怎么对go去做监控呢？指标采集流程说一下？
 24. 你平常有云服务使用经验吗？如AWS或者google？

### <span class="ez-toc-section" id="%E5%85%B6%E5%AE%83"></span>其它<span class="ez-toc-section-end"></span> {.wp-block-heading}

  1. 你们用git作为项目的开发流程是怎么样的？（就是切分支，合分支的流程）
  2. 你理想中的公司的公司文化是怎么样的？你的同事是怎样的一群人？
  3. 有什么想问我的吗？

### <span class="ez-toc-section" id="%E7%AE%97%E6%B3%95"></span>算法<span class="ez-toc-section-end"></span> {.wp-block-heading}

  1. 环形链表（https://leetcode.cn/problems/linked-list-cycle-ii/description）（面试官说不需要写代码，说出解题思路就行）