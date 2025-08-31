---
title: 根据离散函数概率返回int值
date: 2019-01-22T00:58:06+00:00
categories:
  - 算法与数据结构
---
<figure class="wp-block-image"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/01/概率.png" alt="" class="wp-image-375" /><figcaption>图1.1 左侧（期望结果）；右侧（实现的算法代码）</figcaption></figure> 

笔者在阅读《算法》一书的时候看到这样一道示例题，刚开始没有想清楚，而后结合一些帖子和高数的知识便将算法想要传达的意思想明白了，下边我会从均匀分布函数开始记录笔者思考的过程。<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2019/01/均匀分布.png" alt="" class="wp-image-376" /> <figcaption>图1.2 均匀分布</figcaption></figure> 

简单回顾均匀分布函数的知识，更多详情资料读者需自行查阅。 均匀分布也叫矩形分布，是指任意相同间隔所对应的概率分布都相等，该分布有两个参数：最小值(a)和最大值(b)，缩写为U(a, b)。函数为：<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2019/01/均匀分布函数.png" alt="" class="wp-image-379" /> </figure> <figure class="wp-block-image"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/01/均匀分布函数_2-1.png" alt="" class="wp-image-381" /></figure> 

当a=0,b=1时，为标准均匀分布。 对Java语言的Random库，似乎只能产生服从正态分布_N_(0,1)和均匀分布_U_(0,1)的随机数，那么如何按照特定的概率生成随机数呢？<figure class="wp-block-table aligncenter is-style-stripes">

<table>
  <tr>
    <td>
      X
    </td>
    
    <td>
    </td>
    
    <td>
      1
    </td>
  </tr>
  
  <tr>
    <td>
      P(X = x)
    </td>
    
    <td>
      p
    </td>
    
    <td>
      1 &#8211; p
    </td>
  </tr>
</table></figure> 

容易想到，对于服从均匀分布_U_(0,1)的随机变量，产生的随机数落在[0,p)的概率就为p, 而落在[p,1)的概率为1−p。  
接下来，我们考虑稍微复杂的情况： <figure class="wp-block-table aligncenter is-style-stripes">

<table>
  <tr>
    <td>
      X
    </td>
    
    <td>
    </td>
    
    <td>
      1
    </td>
    
    <td>
      2
    </td>
  </tr>
  
  <tr>
    <td>
      P(X = x)
    </td>
    
    <td>
      p0
    </td>
    
    <td>
      p1
    </td>
    
    <td>
      p2
    </td>
  </tr>
</table></figure> 

产生的随机数落在[0,p0​)的概率就为p0​, 而落在[p0​,p0​+p1​)的概率为p1​，落在[p0​+p1​,1)，也就是落在[p0​+p1​,p0​+p1​+p2​)的概率为p2。以此类推，我们可以归纳出以下结论： <figure class="wp-block-table aligncenter is-style-stripes">

<table>
  <tr>
    <td>
      x
    </td>
    
    <td>
    </td>
    
    <td>
      1
    </td>
    
    <td>
      2
    </td>
    
    <td>
      &#8230;
    </td>
    
    <td>
      i
    </td>
    
    <td>
      &#8230;
    </td>
    
    <td>
      n
    </td>
  </tr>
  
  <tr>
    <td>
      P(X = x)
    </td>
    
    <td>
      p0
    </td>
    
    <td>
      p1
    </td>
    
    <td>
      p2
    </td>
    
    <td>
      &#8230;
    </td>
    
    <td>
      pi
    </td>
    
    <td>
      &#8230;
    </td>
    
    <td>
      pn
    </td>
  </tr>
</table></figure> 

产生的随机数落在[∑k=0i−1​ pk​,∑k=0i​ pk​)的概率为pi​，由此我们可以考虑对p进行累加来求解问题，如图1.1。

结合均匀分布函数理解代码：  
产生一个[0, 1]区间均匀分布的随机数r，若

<pre class="wp-block-preformatted">r在[0, a[0])区间，x取0<br />r在[a[0], a[0] + a[1])区间，x取1； <br />r在[a[0] + a[1], a[0] + a[1]  + a[2])区间 ，x取2； <br />...<br />r在[a[0] + a[1]  + ... + a[n-2], 1)区间（相当于 [a[0] + a[1]  + ... + a[n-2], [a[0] + a[1]  + ... + a[n-2] + a[n-1] ) ） ，x取n-1。</pre>

由此，有P( X = i) = a\[i\] (i = 1, 2, 3, &#8230;, n-1)  


参考资料：

  * <http://www.cnblogs.com/yangxiaoling/p/9494504.html>
  * <https://blog.csdn.net/baidu_41324827/article/details/83989618>
  * <https://blog.csdn.net/u013781952/article/details/41950979>