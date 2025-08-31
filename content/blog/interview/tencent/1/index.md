---
title: 腾讯社招一面
date: 2025-01-04T07:10:57+00:00
categories:
  - 面经
---

### <span class="ez-toc-section" id="%E8%83%8C%E6%99%AF"></span>背景<span class="ez-toc-section-end"></span> {.wp-block-heading}

  * 公司：腾讯
  * 部门：区块链产品中心
  * 第几面：一面
  * 方向：后端开发
  * 语言：Go
  * 投递渠道：Boss
  * 面试时间：2024.10.6 9:30
  * 面试方式：线上（腾讯会议）

### <span class="ez-toc-section" id="%E6%80%BB%E4%BD%93%E6%83%85%E5%86%B5"></span>总体情况<span class="ez-toc-section-end"></span> {.wp-block-heading}

面试持续拷打了2小时30分钟，00:00-00:40聊第一间公司的项目，00:40-01:00聊第二间公司的项目，01:00-01:35问八股，01:35-01:50聊其它，01:50-2:30写算法。果然跟网上传言的“有时候腾讯面试时间会很长”一样。

腾讯面试的风格有点八股战士的感觉，问了挺多八股（相对于我这次面试的其它公司来说），而且八股的种类也挺齐全的了（编程语言、数据库、缓存、网络编程、操作系统）。

因为面试的岗位跟我第一间公司业务相似，所以第一间公司的项目也问了很多。

### <span class="ez-toc-section" id="%E9%A1%B9%E7%9B%AE"></span>项目<span class="ez-toc-section-end"></span> {.wp-block-heading}

#### <span class="ez-toc-section" id="%E4%BB%8B%E7%BB%8D%E7%AC%AC%E4%B8%80%E9%97%B4%E5%85%AC%E5%8F%B8%E7%9A%84%E9%A1%B9%E7%9B%AE"></span>介绍第一间公司的项目<span class="ez-toc-section-end"></span> {.wp-block-heading}

追问项目背景、团队角色、业务流程、方案设计、实现细节、技术原理、系统性能

#### <span class="ez-toc-section" id="%E7%AE%80%E5%8D%95%E4%BB%8B%E7%BB%8D%E7%AC%AC%E4%BA%8C%E9%97%B4%E5%85%AC%E5%8F%B8%E7%9A%84%E9%A1%B9%E7%9B%AE"></span>简单介绍第二间公司的项目<span class="ez-toc-section-end"></span> {.wp-block-heading}

追问项目架构、部署方式、技术难/亮点、缓存实现、数据库优化

### <span class="ez-toc-section" id="%E5%85%AB%E8%82%A1"></span>八股<span class="ez-toc-section-end"></span> {.wp-block-heading}

  1. 对k8s熟悉吗？
  2. wait group的三个方法说一下？
  3. 多个defer是按照什么顺序执行的？
  4. select用于什么场景？它的执行流程，比如随机性？
  5. channel底层数据结构是什么样的？
  6. channel被关闭后，继续读/写会怎么样？
  7. go的内存泄漏和性能调优有经验吗？遇到过什么问题？
  8. go的内存分配说一下
  9. go的GC流程说一下
 10. TCP四次挥手流程说一下
 11. TCP滑动窗口是做什么的？
 12. HTTP从1.0到2.0的演化过程中解决了什么问题？
 13. webSocket有了解吗？
 14. 介绍一下进程和线程
 15. 操作系统里面的内核态、用户态怎么理解？
 16. 虚拟内存是干什么的？
 17. 文件从写到磁盘的流程？先写到缓存再定时刷盘？还是立即刷盘？
 18. Redis有哪几种数据形式
 19. Redis如何做到高可用？
 20. MySQL的高可用说一下
 21. https的流程说一下
 22. 对称和非对称加密说一下
 23. go读写锁重复释放会怎样？

### <span class="ez-toc-section" id="%E5%85%B6%E5%AE%83"></span>其它<span class="ez-toc-section-end"></span> {.wp-block-heading}

  * 说一下你们现在的开发节奏和流程
  * 看一下你现在写的项目代码和提交记录（非泄密的）
  * 你有什么想问我的？（通过面试官的回答已经能感觉到通过一面的概率挺高） 
      * 我问：您觉得我面试情况怎么样？有什么缺点需要改正吗？（面试官这里已经明着跟我说对我比较满意了）
      * 我问：公司规模怎么样？有多少人呢？（有很认真的介绍他们部门的背景和人数规模）
  * 你现在在哪里？ 因为base和你现在城市不一样，是有考虑后面会过来发展吗？手里还有其他offer吗？
  * 趋于什么考虑在第二份工作换了方向？（因为我第一家和第二家公司业务不同）

### <span class="ez-toc-section" id="%E7%AE%97%E6%B3%95"></span>算法<span class="ez-toc-section-end"></span> {.wp-block-heading}

  1. 用三个携程，轮流打印cat dog fish，要求顺序不能变化
  2. 反转链表（https://leetcode.cn/problems/reverse-linked-list/description）