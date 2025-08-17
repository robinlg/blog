---
title: Redis之应用解决方案记录
date: 2020-03-22T14:51:42+00:00
categories:
  - Redis
tags:
  - redis solutions

---
本文内容仅为个人记录，并未记录书中完整内容。更详细的内容请查看本文引用的书籍，书名请查看文末。

### <span class="ez-toc-section" id="1_%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81"></span>**1. 分布式锁**<span class="ez-toc-section-end"></span> {.wp-block-heading}

分布式应用进行逻辑处理经常会遇到并发问题。一个操作要修改用户的状态，修改状态需要先读出用户的状态，在内存里进行修改，改完了再存回去。  
  
如果这样的操作同时进行，就会出现并发 问题，因为“读取”和“保存状态”这两个操作不是原子操作。这个时候就要使用到分布式锁来限制程序的并发执行

#### <span class="ez-toc-section" id="11_%E8%B6%85%E6%97%B6%E9%97%AE%E9%A2%98"></span>**1.1 超时问题**<span class="ez-toc-section-end"></span> {.wp-block-heading}

Redis的分布式锁不能解决超时问题，如果在加锁和释放锁之间的逻辑执行太长，以至于超出了锁的限制，就会出现问题。为了避免这个问题，Redis分布式锁不要用于较长时间的任务。如果真的偶尔出现了问题，造成的数据小错乱可能需要人工介入解决。  
  
有一个稍微安全一点的方案是匹配随机参数value和删除key，但这不是一个原子操作，Redis也没提供类似的原子操作指令，这时就需要用Lua脚本处理。但这也不是一个完美的方案，只是相对安全一点，因为如果真的超时了，当前线程逻辑没有执行完，其他线程也会乘虚而入。

### <span class="ez-toc-section" id="2_%E5%BB%B6%E6%97%B6%E9%98%9F%E5%88%97"></span>**2. 延时队列**<span class="ez-toc-section-end"></span> {.wp-block-heading}

Rabbitmq和Kafka这两个都是专业的消息队列中间件，特性之多超出了大多数人的理解能力。对于那些只有一组消费者的消息队列，使用Redis可以非常轻松地搞定，需要注意，Redis不是专业的消息队列，没有非常多的高级特性，没有ack保证，如果对消息的可靠性有着极高要求，那么他就不适用。

#### <span class="ez-toc-section" id="21_%E5%BC%82%E6%AD%A5%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97"></span>**2.1 异步消息队列**<span class="ez-toc-section-end"></span> {.wp-block-heading}

Redis的list常用来作为异步消息队列使用，用rpush和lpush操作入队列，用lpop和rpop操作出队列。

#### <span class="ez-toc-section" id="22_%E9%98%9F%E5%88%97%E7%A9%BA%E4%BA%86%E6%80%8E%E4%B9%88%E5%8A%9E%EF%BC%9F"></span>**2.2 队列空了怎么办？**<span class="ez-toc-section-end"></span> {.wp-block-heading}

如果队列空了，客户端就会陷入pop的死循环，不停pop，没有数据。空轮询不但拉高客户端CPU消耗，Redis的QPS也会被拉高。有以下两种种解决办法。

  1. 使用sleep来解决问题，睡个1S就可以了。不但客户端CPU消耗能降下来，Redis的QPS也能降下来；
  2. 如果有多个消费者，上面的就会导致消息延迟增大。使用阻塞读就是更好的解决方案，阻塞读在队列没数据时会立即进入休眠状态，一旦数据进来，就会立即醒来，消息的延迟几乎为零。blpop/brpop替代lpop/rpop。

阻塞读有可能造成空闲连接问题，所以编写客户端消费者时，如果捕获到异常，还要重试。

#### <span class="ez-toc-section" id="23_%E9%94%81%E5%86%B2%E7%AA%81%E5%A4%84%E7%90%86"></span>**2.3 锁冲突处理**<span class="ez-toc-section-end"></span> {.wp-block-heading}

客户端在处理请求加锁没成功怎么办？一般有以下三种策略来处理加锁失败。

  1. 直接抛出异常，通知客户稍后重试；
  2. sleep一会，然后再重试；
  3. 将请求转移至延时队列，过一会再试。

### <span class="ez-toc-section" id="3_%E4%BD%8D%E5%9B%BE"></span>**3. 位图**<span class="ez-toc-section-end"></span> {.wp-block-heading}

平常开发中，会有一些bool型数据需要存取，比如用户一年的签到记录，签了是1，没签是0，要记录365天。如果使用普通的key/value，当用户上亿时，需要的储存空间是惊人的。为了解决这个问题，Redis提供了位图的数据结构，这样每天签到记录只占一个位，365个为，46个字节就完全可以容纳。

### <span class="ez-toc-section" id="4_HyperLogLog"></span>**4. HyperLogLog**<span class="ez-toc-section-end"></span> {.wp-block-heading}

如果要统计网站上每个网页每天的UV数据，如何实现？

如果统计PV，给每个网页配一个独立的Redis计数器就可以，把这个计数器的key后缀加上当天的日期。这样，执行incrby指令一次即可统计所有PV数据。

UV需要去重，同一个用户一天之内的多次访问请求只能计数一次。Redis提供的HyperLogLog数据结构就是用来解决这种统计问题的。HyperLogLog提供不精确的去重计数方案，虽然不精确，但是不是非常离谱，标准误差是0.81%。这种精确度已经可以满足上面的疑问了。

### <span class="ez-toc-section" id="5_%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8"></span>**5. 布隆过滤器**<span class="ez-toc-section-end"></span> {.wp-block-heading}

如果我们想知道某一个值是否在HyperLogLog结构里面，它就无能为力。

其中一个使用场景：比如在使用新闻客户端看新闻时，它会不停给我们推荐新内容，而每次推荐时都要去重，以去掉那些我们已经看过的内容。新闻客户端推荐系统是如何实现推送去重的？

高级数据结构布隆过滤器（Bloom Filter）就是专门用来解决这种去重问题。它在起到去重作用同时，在空间上还能节省90%以上，只是稍微有一点不准确，也就是有一定的误判概率。

### <span class="ez-toc-section" id="6_%E7%AE%80%E5%8D%95%E9%99%90%E6%B5%81"></span>**6. 简单限流**<span class="ez-toc-section-end"></span> {.wp-block-heading}

系统要限定用户的某个行为在指定的时间里只能允许发生N次，如何使用Redis的数据结构来实现这个限流功能？

这个限流需求中存在一个滑动时间窗口，可以通过zset数据结构的score来圈出这个时间窗口，只需要保留这个时间窗口，窗口之外的数据都可以砍掉。

同一个zset结构记录用户的行为历史，每一个行为都会作为zset中key保存下来。同一个用户同一种行为用一个zset记录。

为节省内存，只需要保留时间窗口内的行为记录，同时如果用户是冷用户，滑动时间窗口内的行为是空记录，那么这个zset就可以从内存中移除，不再占用空间。

书里代码的整体思路是：每一个行为到来时，都维护一次时间窗口。将时间窗口外的记录全部清理掉，只保留窗口内的记录。这几个连续的Redis操作都是针对同一个key的，使用pipeline可以显著提升Redis存取效率。但缺点是量大的时候，如限定60s内操作不得超过100w次这样的参数，是不适合做这样的限流的，因为会消耗大量的存储空间。

### <span class="ez-toc-section" id="7_%E6%BC%8F%E6%96%97%E9%99%90%E6%B5%81"></span>**7.&nbsp;漏斗限流**<span class="ez-toc-section-end"></span> {.wp-block-heading}

Redis4.0提供了一个限流Redis模块，叫redis-cell。该模块使用了漏斗算法，并提供了原子的限流指令。

### <span class="ez-toc-section" id="8_GeoHash"></span>**8. GeoHash**<span class="ez-toc-section-end"></span> {.wp-block-heading}

Redis在3.2版本以后增加了地理位置GEO模块，意味着可以使用Redis实现“附近的人”这样的功能了。

业界比较通用的地理位置距离排序算法是GeoHash算法，Redis也使用GeoHash算法。GeoHash算法将二维的经纬度数据映射到一维的整数，这样所有的元素都将挂载到一条直线上。

映射算法是将地球看成一个二维平面，然后划分成一系列正方形的方格，所有元素坐标都放置于唯一方格中。方格越小，坐标越精细。然后对这些方格进行整数编码，越是靠近的方格编码越是接近。一个最简单的方案是二分法，一个正方形，二刀下去均分成四个小正方形，这时候四个小正方形分别标记为00，01，10，11四个二进制数，然后对每个小正方形继续用二刀法切割，这时候每个小小正方形就可以使用46bit的二进制整数予以表示。继续切下去，二进制整数会越来越长，精确度就会越来越高。除了二刀法，真实算法中还有很多其他刀法，最终编码出来的整数也不一样。

### <span class="ez-toc-section" id="9_Scan"></span>**9.&nbsp;Scan**<span class="ez-toc-section-end"></span> {.wp-block-heading}

Redis在2.8版本中加入了大海捞针的指令——scan。它解决了：

  * 没有offset，limit参数，一次性吐出所有满足条件的key；
  * keys算法是遍历算法，复杂度是O(n)，如果实例中有千万级以上的key，这个指令就会导致Redis服务卡顿。

scan相比keys具备以下特点：

  * 复杂度虽然也是O(n)，但它是通过游标分布进行测，不会阻塞线程；
  * 提供limit参数，可以控制每次返回结果的最大条数；
  * 同keys一样，它也提供匹模式匹配功能；
  * 服务器不需要为游标保存状态，游标的唯一状态就是scan返回给客户端的游标整数；
  * 返回的结果可能会有重复，需要客户端去重复，这点非常重要；
  * 遍历的过程中如果有数据修改，改动后的数据能不能遍历到是不确定的；
  * 单词返回的结果是空的并不意味着遍历结束，而要看返回的游标值是否为零。

在平时的业务开发中，要尽量避免大key的产生。如果观察到Redis的内存大起大落，这极有可能是因为大key导致的，这时候就需要定位具体是哪个key，进一步定位出具体的业务来源，然后再改进相关业务代码涉及。

Redis官方已经在redis-cli指令中提供了扫描功能，可以直接拿来用。

<p class="has-vivid-cyan-blue-color has-text-color">
  文章参考来源《Redis深度历险 核心原理与应用实践》——钱文品
</p>