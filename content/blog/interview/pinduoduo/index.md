---
title: 拼多多社招一面
date: 2025-01-11T06:41:06+00:00
categories:
  - 面经
summary: 拼多多云平台运维后端开发面试经验，主要考察算法题和Go并发编程，面试官偏向运维方向，对开发项目关注度不高。
---

### <span class="ez-toc-section" id="%E8%83%8C%E6%99%AF"></span>背景<span class="ez-toc-section-end"></span> {.wp-block-heading}

  * 公司：拼多多
  * 部门：云平台运维
  * 第几面：一面
  * 方向：后端开发
  * 语言：Go
  * 投递渠道：Boss
  * 面试时间：2024.11.12 20:00
  * 面试方式：线上（拼多多自研的浏览器面试工具）

### <span class="ez-toc-section" id="%E6%80%BB%E4%BD%93%E6%83%85%E5%86%B5"></span>总体情况<span class="ez-toc-section-end"></span> {.wp-block-heading}

面试时长30分钟，因为面的职位偏运维，而我之前做的项目是偏开发，所以面试官在简单聊了下第一个项目之后就让做两个算法题就结束了，没有问八股。

我估计是对我的项目不感兴趣吧，和他们运维职位需要的经验不是很匹配。而且能感受到面试官的业务思维也是一般，估计因为他日常是做运维工作。

### <span class="ez-toc-section" id="%E9%A1%B9%E7%9B%AE"></span>项目<span class="ez-toc-section-end"></span> {.wp-block-heading}

简单聊了下业务流程和一些技术实现细节

### <span class="ez-toc-section" id="%E7%AE%97%E6%B3%95"></span>算法<span class="ez-toc-section-end"></span> {.wp-block-heading}

1. 打印出二叉树最大深度的路径

<pre class="wp-block-code"><code lang="go" class="language-go">var nums = []int{1,2,3,4,5}
var res []int

for i := range nums {
  go func() {
    res = append(res, nums[i])
  }
}
fmt.Println(res)</code></pre>

2. 打印出二叉树最大深度的路径