---
title: 区块链服务网络BSN方案分享
date: 2020-02-29T10:14:41+00:00
categories:
  - 区块链
tags:
  - BSN

---
最近笔者参加了BSN这一项赛事，随着赛事的完毕，现将此次个人的作品方案出来。由于参赛的前提是参赛者必须同意其方案可以向公众无偿展示，笔者就无谓在博客分享该方案。但其版权和署名权均归原作者（即笔者）所有，欢迎转载，但请告知。

此次赛事无须敲代码，只需写业务流程、软件架构等模块的方案。笔者感觉更适合架构师，或者产品经理参加此次赛事。笔者自认还没达到架构师的水平，但还是想将内心的想法表达出来，就将思路以相对简单的文档表达出来并提交了，全文如下：

## <span class="ez-toc-section" id="%E3%80%8A%E5%9F%BA%E4%BA%8E%E5%9C%B0%E5%9B%BE%E8%BD%AF%E4%BB%B6%E7%9A%84%E9%93%BE%E6%94%B9%E6%96%B9%E6%A1%88%E3%80%8B"></span>**《基于地图软件的链改方案》**<span class="ez-toc-section-end"></span> {.has-text-align-center.wp-block-heading}

<p class="has-text-align-center has-regular-font-size">
  <strong>2020/2/14</strong><br /><strong>Robin</strong> <strong><br /></strong>
</p>

### <span class="ez-toc-section" id="%E5%89%8D%E8%A8%80"></span>前言  
<span class="ez-toc-section-end"></span> {.has-text-align-center.wp-block-heading}

在最终敲定该方案前，笔者也曾构思过其它方案。或是因业务不熟悉且流程复杂，又或是因不能实际解决问题，最终没被笔者采纳。但这并不代表本方案完美无瑕，在笔者看来该方案存在的最大问题就是商业合作，似乎各方不能直接从中获得利益，但从“老百姓”的角度来看，实属一件惠民之事。  
  
该想法曾在去年和公司的业务提起，无奈业务对“目前看不到利益”的事情不感兴趣，也因为公司力量有限，该想法一直被埋在心底。本方案在“现状与痛点”的章节从软件使用者的角度提出疑问及期待，利益方的盈利方式思考甚少。  
  
曾经笔者也是“唯链不可”的人，经过一段时间的洗涤与沉淀，现在笔者认为能解决大众生活问题的应用才是好应用。也正逢此次机会，将我个人的想法描述出来，或许所述机构能从本方案中看到笔者所看不到的利益点，次之，有眼光长远的读者能在此之上添砖加瓦，普罗大众。  
  
归根结底，想将区块链用在真正解决问题之处。至于其中的利益分配，就转由更专业的人士去分析罢。 

### <span class="ez-toc-section" id="%E7%8E%B0%E7%8A%B6%E5%8F%8A%E7%97%9B%E7%82%B9"></span><a>现状及痛点</a><span class="ez-toc-section-end"></span> {.has-text-align-center.wp-block-heading}

随着现代人民的生活水平逐渐提高，各式各样的娱乐消费场所也越开越多。对于绝大多数消费者来说，去一趟从未涉足过的场所，几乎都会使用手机地图导航。如：高德地图、百度地图、腾讯地图等。而现在负责各地图软件营运的公司处于相互竞争的状态，地标数据都存自身的数据库中，导致出现同一处场所在A地图软件中出现而B地图软件中没有出现的情况。  
  
遇此情况使用者还需下载另一个地图软件，若另一个地图软件也缺少该地标信息，可能还需要再重复下载。笔者站在使用者角度来思考，营运各地图软件的公司应该基于这些齐全的地标信息去完善自身提供的服务、功能，提升自身软件的服务价值，从而获得更多的盈利。  
  
笔者从两个不同的地图软件中截图，由图可见地图软件B没有“灯塔咖啡”这个地标。 

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2020/02/Screenshot_20200128_153407_com.baidu_.BaiduMap-e1582970291108-692x1024.jpg" alt="" class="wp-image-520" width="346" height="512" /><figcaption>地图软件A</figcaption></figure>
</div>

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2020/02/Screenshot_20200128_153419_com.autonavi.minimap-e1582970503572-692x1024.jpg" alt="" class="wp-image-521" width="346" height="512" /><figcaption>地图软件B</figcaption></figure>
</div>

###  {.has-text-align-center.wp-block-heading}

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2020/02/流程图-752x1024.png" alt="" class="wp-image-522" width="376" height="512" /><figcaption><a>“链改”前的业务流程图</a> </figcaption></figure>
</div>

###  {.has-text-align-center.wp-block-heading}

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2020/02/未命名文件-897x1024.png" alt="" class="wp-image-523" width="449" height="512" /><figcaption> <a>“链改”前的系统结构图</a> </figcaption></figure>
</div>

各地图软件的技术实现细节无法深入调研，只能大致画出如上图所示的三层结构，各方的数据存在各自的数据库中，没有进行数据交换。

###  {.has-text-align-center.wp-block-heading}

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2020/02/链改前-1013x1024.png" alt="" class="wp-image-524" width="507" height="512" /><figcaption> <a>“链改”后的系统结构图</a> </figcaption></figure>
</div>

地图软件营运各方搭建联盟链节点，形成一条共享地图标点信息的联盟链。审核通过后的标点信息入链。  
  
标点信息包括：1. 地址；2. 经纬度；3. 联系电话（若有）；4. 商铺（场所）名称等。 

### <span class="ez-toc-section" id="%E6%94%B9%E9%80%A0%E5%89%8D%E5%90%8E%E7%9A%84%E5%AF%B9%E6%AF%94"></span><a>改造前后的对比</a>  
<span class="ez-toc-section-end"></span> {.has-text-align-center.wp-block-heading}

**改造前**

  1. 每个地图软件或多或少都会缺失某些场所标点，这些缺失的标点往往能在另一个地图软件中找到。给用户的使用上带来不便，习惯了某一地图软件的用户可能为了找一处标点而下载另一个地图软件。即便自身的服务（例如导航、实景地图等）做得很好，也可能因为标点的缺失而流失用户；
  2. 各地图软件营运方可能因缺少的这些标点的信息，无法做到精准推荐或者算法参数的缺失；
  3. 若政府部门有获取齐全地标的需求，按照现有的系统，需要分别调取各地图软件营运方的数据库所有标点信息，流程上较复杂，且标点信息重复率极高，需要进行筛选后才能得知哪些标点不是各方数据库都存在。

**改造后**

  1. 各地图软件的营运方审核通过用户上报的地点后，将标点信息入链，节点各方互享标点信息，并显示在各自地图软件上。标点信息（地址、经纬度、联系电话（若有）、名称）都是在地图上开放给公众给查看的，所以不需要加密入链，并且这些信息所占字节数很少，适合节点间传输、共识；
  2. 各地图软件营运方可以根据齐全的标点信息，提升自身的服务、功能，从而提高地图软件的用户友好性，凭借更好的服务吸引用户；
  3. 相关政府部门仅需在自身服务器部署联盟链节点，并借入已组建好的区块链网络即可查看齐全的标点信息。

### <span class="ez-toc-section" id="%E8%8A%B1%E7%B5%AE"></span>花絮<span class="ez-toc-section-end"></span> {.has-text-align-center.wp-block-heading}

去年，笔者出差澳门，中午在茶餐厅吃饭，发现一些大陆不常见的事：一些澳门的长辈喜欢拼桌与看起来有趣的人聊上一小会便离去。那天与我搭话的是一个工作在机场控制塔台的长辈，大约四十来岁。饶有兴致地问我的工作和此行的目的，简短交谈后发现，长辈竟然也知道区块链。“我闲下来的时候还在想区块链能不能用在控制塔台上呢，因为一架飞机从我们这里起飞，要同时将航班信息发给许多别的塔台……我要赶去上班了，再见。”离开茶餐厅后，我也在想一个塔台能不能作为一个节点？晚上工作结束后发现这里面的公开资料实在太少，于是暂时放弃了这个想法。时隔一年，有了此次BSN的平台，我将这个有趣的发现放在“花絮”部分里，说不定能激起读者们的遐想。