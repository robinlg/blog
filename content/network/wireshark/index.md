---
title: 使用抓包工具看网络分层
date: 2021-05-01T03:14:07+00:00
categories:
  - 网络
tags:
  - wireshark

---
在学习小册<a rel="noreferrer noopener" href="https://juejin.cn/book/6844733788681928712" target="_blank">《深入理解TCP协议：从原理到实战》</a>的时候发现一个不错的抓包工具“**wireshark**”，借此篇文章做一下简介，顺便记录学习过程。<figure class="wp-block-image is-resized">

<img decoding="async" loading="lazy" src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97e2c53950e14115af6e685b2bb481a7~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" width="836" height="705" /> </figure> 

我将分层和一些主要参数的解释都表达在图片上了，就不再另作文字说明。通过抓包分析出来的信息，引用小册里的一段话：

<blockquote class="wp-block-quote">
  <p>
    纵观计算机和分布式系统，你会发现「计算机的问题都可以通过增加一个虚拟层来解决，如果不行，那就两个」
  </p>
</blockquote>

PS：以上是我在掘金写的文章，所以图片带水印。