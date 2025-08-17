---
title: 哥尼斯堡七桥问题
date: 2019-03-02T09:23:27+00:00
categories:
  - 算法与数据结构
tags:
  - 奇偶性校验
---

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/03/QU@YRW40DMTMCYEJW8H-1.png" alt="" class="wp-image-409" /><figcaption>图1</figcaption></figure>
</div>

要求：现在你要找出走遍七座桥的方法。但是必须遵守以下条件：

  * 走过的桥不可再走
  * 可以经过同一陆地
  * 可以以任一陆地为起点
  * 不需回到原点

若能走遍七座桥，请说明方法。若不能，也请给出证明。

其实这就是“一笔画”的问题，意思就是从下笔开始，所画的线要经过每一块陆地，笔画不能出现间断。用图2的方法发现最后也到达不了g桥。读者们也可以多多画一下，尝试一下。最后发现根本不可能走遍七座桥，但是我们要证明出来。因为或许有方法只是我们没有想到而已。  


<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/03/@OZV9N4PCRBNZ5LLSUH.png" alt="" class="wp-image-410" /><figcaption>图2</figcaption></figure>
</div>

我们考虑将问题简化一下，因为每次使用上图来进行试验以及问题的思考其实是相对麻烦的。这种图形化的“链接方式”称作“图”（Graph）。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/03/WDV7GPTLFTAGR8LG.png" alt="" class="wp-image-411" /><figcaption>图3</figcaption></figure>
</div>

在图3中，用白色的圈来表示陆地A、B、C、D，我们成为“顶点”。用顶点之间的连线来表示桥a、b、c、d、e、f、g，我们称其为“边”。

顶点所关联的边数称其为“度数”。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/03/Y3P95C8PCLEUAP.png" alt="" class="wp-image-412" /><figcaption>图4</figcaption></figure>
</div>

度数为偶数的点称为“偶点”，度数为奇数的点称为“奇点”。接下来顺着图中的边走，在经过的边的端点出打上钩，并减去顶点的度数。我们将此称为“边走边减”。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/03/V90EYEFF3GKH48AKC.png" alt="" class="wp-image-413" /><figcaption>图5</figcaption></figure>
</div>

目前不关心具体是从哪里开始的，经过的路径，只看顺着边走的时候顶点的度数是如何变化的。出发时，起点的度数减1。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/03/O4ZDUO6FWG@J_RFMG.png" alt="" class="wp-image-414" /><figcaption>图6</figcaption></figure>
</div>

途中经过每一个顶点时，该顶点的度数减2，因为经过了“入口边”和“出口边”。  


<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/03/KX7TR2SC3ZBU1YBZA.png" alt="" class="wp-image-415" /><figcaption>图7</figcaption></figure>
</div>

每次经过顶点，顶点的度数都会减2。因此不管经过顶点多少次，经过的顶点的奇偶性不会变，即奇点还是奇点，偶点还是偶点。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/03/AVH26IKREEHTWDMBEFQ.png" alt="" class="wp-image-416" /><figcaption>图8</figcaption></figure>
</div>

到达终点时，该顶点的度数减1。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/03/QK9S@7R9HP_9@X8WJ-1.png" alt="" class="wp-image-418" /><figcaption>图9</figcaption></figure>
</div>

我们假设如此“完成了一笔画”，那么可能出现一下两种情况。

（1）起点和终点相同的情况  
&nbsp;&nbsp;&nbsp;&nbsp;一笔画成，也就意味着“边走边减”的结果是所有顶点的度数变为0（偶数）。为什么？因为如果还存在不为0的顶点，那么也就存在没经过的边。  
&nbsp;&nbsp;&nbsp;&nbsp;经过“边走边减”之后，经过的顶点的奇偶性不变。由此我们可知度数变为0（偶数）的经过点，在原图中本身就是偶点。  
&nbsp;&nbsp;&nbsp;&nbsp;此外，起点度数减1，终点度数也减1，变为0。然而，起点和终点是同一顶点，所以顶点的度数减2，所以该顶点也成了偶点。  
&nbsp;&nbsp;&nbsp;&nbsp;结论，在“起点与终点相同”的一笔画中，图中所有的顶点都是“偶点”。

（2）起点和终点不相同的情况  
&nbsp;&nbsp;&nbsp;&nbsp;和（1）的 思路相同，经过的顶点都是偶点，只有起点和终点是奇点。据此，在“起点和终点不同”的一笔画中，图中只有两个“奇点。  
&nbsp;&nbsp;&nbsp;&nbsp;至此，可知以下命题是成立的：

（如果）“可以画成一笔画” =》“所有顶点都是偶点，或者有2个奇点。”

我们回到哥尼斯堡七桥问题。如果哥尼斯堡的七桥能用一笔画通过的话，应该满足 “ 所有顶点都是偶点，或者有2个奇点 ”。

我们来看看哥尼斯堡七桥（图10）的顶点。如图所示，四个点都是“奇点”。由此证明了在给定条件下不能走遍哥尼斯堡七桥。  


<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/03/PSI43O9V3HV04PZKQK@9.png" alt="" class="wp-image-419" /><figcaption>图10</figcaption></figure>
</div>

&nbsp;&nbsp;&nbsp;&nbsp;理解了欧拉的“如果能够一笔画成，必须满足所有顶点或者有两个奇点”的这一断论。根据这一断论，我们证明了哥尼斯堡七桥是不能走遍的。  
&nbsp;&nbsp;&nbsp;&nbsp;欧拉的断论重点在于：不用反复试验也能证明不能一笔画的问题。不同频繁地试走各种路径，只要观察各顶点的度数就行了。  
&nbsp;&nbsp;&nbsp;&nbsp;另外，欧拉的证明中蕴含着很重要的思维方法。那就是在观察各顶点的边数时，着眼点不在“数的本身”，而是“数的奇偶性”。并不是1条、3条、5条这样分散地思考路径，而是概括为“奇数条”来整体考虑。在一笔画的问题中，这个“奇偶性”是解题的关键。这是奇偶校验的一个例子。  


  


###### 文章参考来源《程序员的数学》