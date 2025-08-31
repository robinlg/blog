---
title: 高级树、AVL树和红黑树
date: 2021-08-21T08:40:39+00:00
categories:
  - 算法与数据结构
---
<blockquote class="wp-block-quote">
  <p>
    覃超的《算法训练营》课程学习已接近尾声，通过这段时间的学习及训练，对数据结构和算法的认知进一步加深。从业务系统的CURD下沉到底层的数据结构和算法，回顾这些知识给我最大的感触是：这些数据结构和算法科学家/专家非常伟大。
  </p>
</blockquote>

今天用一些时间总结课程里关于“树”的知识。读者需要注意的是，对于下文中提到的几种“树”，这里只做简单的介绍、小结。详细的推理和证明建议有兴趣的读者可以查阅详细的资料，本文就不再花费大量篇幅去拷贝各位科学家的论文。提示：本文里有些超链接需要自备科学上网工具才可访问。

## <span class="ez-toc-section" id="%E6%A0%91%EF%BC%88Tree%EF%BC%89"></span>树（Tree）<span class="ez-toc-section-end"></span>

<img decoding="async" src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d64ba53d21e4cb6b191a88bacf9d1fc~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> </figure> 
  * Root：根节点
  * Parent Node：父节点
  * Child Node：孩子节点
  * Left Node：叶子节点
  * Sub-tree：子树

## <span class="ez-toc-section" id="%E4%BA%8C%E5%8F%89%E6%A0%91%EF%BC%88Binary_Tree%EF%BC%89"></span>二叉树（Binary Tree）<span class="ez-toc-section-end"></span>

<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e72b57aaf8774321924269fecc7c5b89~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> </figure> 

二叉树每个父节点最多拥有两个孩子节点，有如下三种遍历顺序：

### <span class="ez-toc-section" id="1_%E5%89%8D%E5%BA%8F%E9%81%8D%E5%8E%86%EF%BC%88Pre-order%EF%BC%89%EF%BC%9A%E6%A0%B9_-%3E_%E5%B7%A6_-%3E_%E5%8F%B3"></span>1. 前序遍历（Pre-order）：根 -> 左 -> 右<span class="ez-toc-section-end"></span> {#2.1.wp-block-heading}

<pre class="wp-block-code"><code lang="java" class="language-java">// sample code
public void preorder(List&lt;Integer&gt; res, Node root) {
    // terminator
    if (root == null) return;
    
    // business
    res.add(root.val);
    preorder(res, root.left);
    preorder(res, root.right);
}
复制代码</code></pre>

### <span class="ez-toc-section" id="2_%E4%B8%AD%E5%BA%8F%E9%81%8D%E5%8E%86%EF%BC%88In-order%EF%BC%89%EF%BC%9A%E5%B7%A6_-%3E_%E6%A0%B9_-%3E_%E5%8F%B3"></span>2. 中序遍历（In-order）：左 -> 根 -> 右<span class="ez-toc-section-end"></span> {#2.2.wp-block-heading}

<pre class="wp-block-code"><code lang="java" class="language-java">// sample code
public void preorder(List&lt;Integer&gt; res, Node root) {
    // terminator
    if (root == null) return;
    
    // business
    preorder(res, root.left);
    res.add(root.val);
    preorder(res, root.right);
}
复制代码</code></pre>

### <span class="ez-toc-section" id="3_%E5%90%8E%E5%BA%8F%E9%81%8D%E5%8E%86%EF%BC%88Post-order%EF%BC%89%EF%BC%9A%E5%B7%A6_-%3E_%E5%8F%B3_-%3E_%E6%A0%B9"></span>3. 后序遍历（Post-order）：左 -> 右 -> 根<span class="ez-toc-section-end"></span> {#2.3.wp-block-heading}

<pre class="wp-block-code"><code lang="java" class="language-java">// sample code
public void preorder(List&lt;Integer&gt; res, Node root) {
    // terminator
    if (root == null) return;
    
    // business
    preorder(res, root.left);
    preorder(res, root.right);
    res.add(root.val);
}
复制代码</code></pre>

## <span class="ez-toc-section" id="%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91%EF%BC%88Binary_Search_Tree%EF%BC%89"></span>二叉搜索树（Binary Search Tree）<span class="ez-toc-section-end"></span> {#3.wp-block-heading}<figure class="wp-block-image">

<img decoding="async" src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/633742c27e874052a53e97cf10d7a881~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> </figure> 

二叉搜索树，也称二叉搜索树、有序二叉树（Ordered Binary Tree）、排序二叉树（Sorted Binary Tree），是指一颗空树或具有下列性质的二叉树：

  1. 左子树上`所有节点`的值均小于它的根节点的值；
  2. 右子树上`所有节点`的值均大于它的根节点的值；
  3. 以此类推：左、右子树也分别为二叉搜索树（这就是重复性）。

注意：二叉搜索树**中序遍历**的结果是升序排列。

下面我们来思考下面这幅图，找出红色节点所有的时间复杂度是多少？<img decoding="async" src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52eb231ef25949dea941ec3c359c2032~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 在上图这个二叉搜索树中找出红色节点所有的时间复杂度是O(log2N)，这里的2为底数，N为节点数量。

如果出现下图的极端情况，退化成了一个线性的结构，那么找出最底部的节点，其时间复杂度是多少？<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/113103d00a1f4712a2403f1d5ce20e7e~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 在上图的线性结构中找出最底部节点的时间复杂度是O(N)，N为节点数量。

从上面的现象可以得出这样的结论：需要保证二维维度，让左右子树节点平衡。

从维基百科<a href="https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FSelf-balancing_binary_search_tree" target="_blank" rel="noreferrer noopener">Self-balancing binary search tree</a>可以详细查看实现了上述结论的几种树。

  * <a href="https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2F2%25E2%2580%25933_tree" target="_blank" rel="noreferrer noopener">2–3 tree</a>
  * <a href="https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FAA_tree" target="_blank" rel="noreferrer noopener">AA tree</a>
  * <a href="https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FAVL_tree" target="_blank" rel="noreferrer noopener">AVL tree</a>
  * <a href="https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FB-tree" target="_blank" rel="noreferrer noopener">B-tree</a>
  * <a href="https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FRed%25E2%2580%2593black_tree" target="_blank" rel="noreferrer noopener">Red–black tree</a>
  * <a href="https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FScapegoat_tree" target="_blank" rel="noreferrer noopener">Scapegoat tree</a>
  * <a href="https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FSplay_tree" target="_blank" rel="noreferrer noopener">Splay tree</a>
  * <a href="https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTango_tree" target="_blank" rel="noreferrer noopener">Tango tree</a>
  * <a href="https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTreap" target="_blank" rel="noreferrer noopener">Treap</a>
  * <a href="https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FWeight-balanced_tree" target="_blank" rel="noreferrer noopener">Weight-balanced tree</a>

## <span class="ez-toc-section" id="AVL%E6%A0%91"></span>AVL树<span class="ez-toc-section-end"></span> {#4.wp-block-heading}

AVL树的每个节点都有其`平衡因子`，以`平衡因子`判断子树是否要通过旋转操作（有四种旋转操作）来进行平衡。

Balance Factor（平衡因子）：某父节点的左子树高度减去它的右子树高度（有时候右子树减去左子树）。平衡因子有三个值可取，分别是`-1`, ``, `1`。如果某个节点的平衡因子不属于这三个值其中之一，就需要对其和其子树进行旋转操作。

下图是一个AVL树，绿色的数字代表其节点的`平衡因子`。<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebf744009b0c4a0ba916bbd34ecc745b~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

当增加一个数值为14的节点，该树的`平衡因子`变为下图。<img decoding="async" src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/354509323b8245f59db03cc59b3b484d~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

当增加一个数值为3的节点，该树的`平衡因子`变为下图。<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65b03251e28d49c788a3b6e09fd02320~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 此时数值为10的节点的`平衡因子`已经不是`-1`, ``, `1`的其中一个了，而是`-2`，就需要对其和其子树进行旋转操作。

如何进行旋转操作？首先介绍四种旋转，最后再对结合上图具体说明使用哪种旋转方法。

### <span class="ez-toc-section" id="1_%E5%B7%A6%E6%97%8B"></span>1. 左旋<span class="ez-toc-section-end"></span> {#4.1.wp-block-heading}

当子树状态为右右子树（某父节点仅有单一的右子树，且该右子树也仅有单一的右子树），就对其进行左旋，如下图（图中左边为右右子树状态，右边为旋转后的状态）。<img decoding="async" src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4899e29803b4237ad0990179815bb49~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

### <span class="ez-toc-section" id="2_%E5%8F%B3%E6%97%8B"></span>2. 右旋<span class="ez-toc-section-end"></span> {#4.2.wp-block-heading}

当子树状态为左左子树（某父节点仅有单一的左子树，且该左子树也仅有单一的左子树），就对其进行右旋，如下图（图中左边为左左子树状态，右边为旋转后的状态）。<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09f0e54a9cfa4e7a9f79da11ebab88da~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

### <span class="ez-toc-section" id="3_%E5%B7%A6%E5%8F%B3%E6%97%8B"></span>3. 左右旋<span class="ez-toc-section-end"></span> {#4.3.wp-block-heading}

当子树状态为左右子树（某父节点仅有单一的左子树，且该左子树仅有单一的右子树），就对其进行左右旋，如下二张图。

首先对左右子树进行左旋，让其成为左左子树。<img decoding="async" src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d1b39c4bf264e4987bd20a887d13695~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

再对左左子树进行右旋。<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b98ca832f3394bc7bb4cbff8cb4c43d5~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

### <span class="ez-toc-section" id="4_%E5%8F%B3%E5%B7%A6%E6%97%8B"></span>4. 右左旋<span class="ez-toc-section-end"></span> {#4.4.wp-block-heading}

当子树状态为右左子树（某父节点仅有单一的右子树，且该左子树仅有单一的左子树），就对其进行右左旋，如下二张图。

首先对右左子树进行右旋，让其成为右右子树。<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f05dad8113bd47229a58aab497e0cae7~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

再对右右子树进行左旋。<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0423fdd0638040bfb966ae4947a46f8a~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

上面介绍的四种旋转操作的简图没将子树画出来。带有子树旋转<a href="https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%23%2Fmedia%2FFile%3ARebalancing.gifhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%2523%2Fmedia%2FFile%3ARebalancing.gifhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%2523%2Fmedia%2FFile%3ARebalancing.gifhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%2523%2Fmedia%2FFile%3ARebalancing.gifhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%2523%2Fmedia%2FFile%3ARebalancing.gifhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%2523%2Fmedia%2FFile%3ARebalancing.gifhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%2523%2Fmedia%2FFile%3ARebalancing.gifhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%2523%2Fmedia%2FFile%3ARebalancing.gifhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%2523%2Fmedia%2FFile%3ARebalancing.gifhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%2523%2Fmedia%2FFile%3ARebalancing.gifhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%2523%2Fmedia%2FFile%3ARebalancing.gifhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%2523%2Fmedia%2FFile%3ARebalancing.gifhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%2523%2Fmedia%2FFile%3ARebalancing.gifhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%2523%2Fmedia%2FFile%3ARebalancing.gifhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FTree_rotation%2523%2Fmedia%2FFile%3ARe" target="_blank" rel="noreferrer noopener">链接1</a>/<a href="https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F63272157" target="_blank" rel="noreferrer noopener">链接2</a>的示意图可参考下图。<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/666481ced9d14ca4ae8c022b4f0b0fc3~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

### <span class="ez-toc-section" id="%E6%97%8B%E8%BD%AC%E6%93%8D%E4%BD%9C%E4%BE%8B%E9%A2%981"></span>旋转操作例题1<span class="ez-toc-section-end"></span> {#4.5.wp-block-heading}

介绍完四种旋转操作后，回到前面提到的一个AVL树：「当增加一个数值为3的节点，该树的`平衡因子`变为下图。」<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65b03251e28d49c788a3b6e09fd02320~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 此时数值为10的节点的`平衡因子`已经不是`-1`, ``, `1`的其中一个了，而是`-2`，就需要对其和其子树进行旋转操作。

因为对数值为10的节点属于左左子树的状态，所以需要对其进行右旋，如下图。<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df60bad7665e498f9a8deb4947460432~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

参考上面带子树旋转的示意图可知，旋转后要将数值为5的节点的子树（8）挂载到数值为10的节点下。旋转后的结果如下图。<img decoding="async" src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62834c12fbcc4b23b015af817d43e1ee~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

### <span class="ez-toc-section" id="%E6%97%8B%E8%BD%AC%E6%93%8D%E4%BD%9C%E4%BE%8B%E9%A2%982"></span>旋转操作例题2<span class="ez-toc-section-end"></span> {#4.6.wp-block-heading}

一个AVL树的`平衡因子`如下图。<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57541b6510494949a50b1c9d475bfbed~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

当增加数值为15的节点，其`平衡因子`变为下图。<img decoding="async" src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64bc454461b4449d98d435e9bb5b7302~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 此时数值为9的节点和数值为7的节点的`平衡因子`已经不是`-1`, ``, `1`的其中一个了，而是`+2`，就需要对其和其子树进行旋转操作。

我们对层数更高的数值为9的节点进行旋转操作，其属于左右子树的状态，所以需要对其进行左右旋，首先对其进行右旋，如下两张图。<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/878fcd0a4d184eb8bfb4fa50d96e1816~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

右旋后的结果如下图。<img decoding="async" src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e9050b95c2b41c79e7085c9d86b2261~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

再对其进行左旋，如下两张图。<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe13d7df392d440db1236f55c4965cc7~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

左旋后的结果如下图。<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebc4895d738a446a904f7dc6a17955fe~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> 

### <span class="ez-toc-section" id="AVL%E6%A0%91%E5%B0%8F%E7%BB%93"></span>AVL树小结<span class="ez-toc-section-end"></span> {#4.7.wp-block-heading}

  1. 它是平衡二叉树;
  2. 每个节点存balance factor = {-1, 0, 1};
  3. 四种旋转操作。

得益于AVL树的数据结构，每个节点的左右子树高度差不超过1，极端境况的时间复杂度也非常接近O(log2N)，因此它的查询效率比较高。但也因此带来了一些不足之处：节点需要存储额外信息（平衡因子），且调整次数频繁，因此出现了红黑树。

## <span class="ez-toc-section" id="%E7%BA%A2%E9%BB%91%E6%A0%91"></span>红黑树<span class="ez-toc-section-end"></span> {#5.wp-block-heading}

红黑树是一种近似平衡的二叉搜索树（BinarySearch Tree），它能够确保任何一个节点的左右子树的高度差小于两倍。具体来说，红黑树是满足如下条件的二叉搜索树：

  * 每个节点要么是红色，要么是黑色
  * 根节点是黑色
  * 每个叶节点（NIL节点，空节点）是黑色的。
  * 不能有相邻接的两个红色节点
  * 从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。

关键性质：从根节点到叶子节点的最长的可能路径不多于最短的可能路径的两倍。<figure class="wp-block-image">

<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc7ef8af99d6422bb7c82b6fb18895d7~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> </figure> <figure class="wp-block-image"><img decoding="async" src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/245cd8f155d647748e677c3227e270d5~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /></figure> 

最后有个AVL树和红黑树的概述和比较，如下图。<figure class="wp-block-image">

<img decoding="async" src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67aae75244f944c7823df0b16bade225~tplv-k3u1fbpfcp-watermark.awebp" alt="image.png" /> </figure> 

  * AVL树的查询效率比红黑树高，因为AVL树更严格的平衡。上面提到的AVL树的**左右子树高度差不超过1**，而红黑树**从根节点到叶子节点的最长的可能路径不多于最短的可能路径的两倍**。
  * 红黑树的插入和删除效率比AVL高，因为红黑树是一个近似平衡树。AVL树如果增加/删除节点会因为`平衡因子`的变化而进行更多的旋转操作。
  * AVL树存的信息要比红黑树更多，所以AVL树额外的消耗空间要比红黑树多。AVL要存`平衡因子`和树的高度，而红黑树只要存一个bit来存0或1表示是黑还是红。
  * 红黑树一般使用在高级语言（Java, C++ &#8230;）里面的map结构比较多，AVL树在数据库里的使用更多。因为根据上面的结论，如果写操作比较多或者读/写操作一半一半的时候，用红黑树即可，所以用在高级语言里的map比较多。而数据库往往是读操作多于写操作，所以一般是用AVL树。

PS：以上是我在掘金写的文章，所以图片带水印。