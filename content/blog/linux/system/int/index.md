---
title: 整数表示
date: 2023-04-22T08:19:40+00:00
categories:
  - Linux
summary: 详细介绍计算机中整数的表示方法，包括补码、反码、原码以及有符号数和无符号数之间的转换、扩展和截取操作。
---
本文为《深入理解计算机系统》— 「2.2 整数表示」章节的小结。

## <span class="ez-toc-section" id="%E8%A1%A5%E7%A0%81"></span>补码<span class="ez-toc-section-end"></span> {.wp-block-heading}

无符号数的编码

![](截屏2023-04-22-下午3.41.27-1.png)

补码编码

![](截屏2023-04-22-下午3.18.36-3.png)

C语言标准并没有要求要用补码形式来表示有符号整数，但是几乎所有的机器都是这么做的。在补码(two&#8217;s -complement )的定义中，将字的最高有效位解释为负权(negative-weight)。

有符号数还有两种表示形式：反码(ones&#8217;-complement )和源码(sign-magnitude)。

![](截屏2023-04-22-下午3.25.16-1.png)

## <span class="ez-toc-section" id="%E6%9C%89%E7%AC%A6%E5%8F%B7%E6%95%B0%E5%92%8C%E6%97%A0%E7%AC%A6%E5%8F%B7%E6%95%B0%E4%B9%8B%E9%97%B4%E7%9A%84%E8%BD%AC%E6%8D%A2"></span>有符号数和无符号数之间的转换<span class="ez-toc-section-end"></span> {.wp-block-heading}

补码转换为无符号数

![](截屏2023-04-22-下午3.47.40-1.png)

无符号数转换为补码

![](截屏2023-04-22-下午3.52.51.png)

在C语言中对同时包含有符号和无符号数表达式的处理时，有需要注意的地方：

![](截屏2023-04-22-下午3.56.25.png)

Go语言就避免了这个可能会犯错的地方。因为Go语言在对同时包含有符号数和无符号数表达式的处理时，不会进行隐式强制转换，需要我们自己进行强制转换。

## <span class="ez-toc-section" id="%E6%89%A9%E5%B1%95%E6%95%B0%E5%AD%97"></span>扩展数字<span class="ez-toc-section-end"></span> {.wp-block-heading}

无符号数的扩展

![](截屏2023-04-22-下午4.02.58.png)

补码数的符号扩展(扩展的位等于补码的最高有效位)

![](截屏2023-04-22-下午4.06.47.png)

## <span class="ez-toc-section" id="%E6%88%AA%E5%8F%96%E6%95%B0%E5%AD%97"></span>截取数字<span class="ez-toc-section-end"></span> {.wp-block-heading}

截取无符号数

![](截屏2023-04-22-下午4.12.44.png)

截取补码数值

![](截屏2023-04-22-下午4.13.32.png)

![](截屏2023-04-22-下午4.14.42.png)

**参考资料**

《深入理解计算机系统第三版》