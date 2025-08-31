---
title: Java并发包工具类使用范式
date: 2021-09-19T08:48:24+00:00
categories:
  - Java
---
<blockquote class="wp-block-quote">
  <p>
    上周看完极客时间的<a href="https://link.juejin.cn?target=https%3A%2F%2Ftime.geekbang.org%2Fcolumn%2Fintro%2F159" target="_blank" rel="noreferrer noopener">《Java并发编程》</a>课程，发现知识讲的通俗易懂，没有很多晦涩的用词，个人认为适合Java并发编程入门的读者。对于技术大牛，就别花这一百元去购买课程了，还是建议看回经典的“圣书”。此篇文章不会写过多关于课程的具体内容，笔者希望有兴趣的读者花钱去购买课程，创作不易，“花钱购买别人的知识”是个良性循环。
  </p>
</blockquote>

以下篇幅仅记录课程中提到的关于并发编程在实际工程中推荐使用的编程范式。**范式，意味着是经典做法，所以没有特殊理由不要尝试换个写法。**


### <span class="ez-toc-section" id="%E7%94%A8%E2%80%9C%E7%AD%89%E5%BE%85-%E9%80%9A%E7%9F%A5%E2%80%9D%E6%9C%BA%E5%88%B6%E4%BC%98%E5%8C%96%E5%BE%AA%E7%8E%AF%E7%AD%89%E5%BE%85"></span><a href="https://link.juejin.cn?target=https%3A%2F%2Ftime.geekbang.org%2Fcolumn%2Farticle%2F85241" target="_blank" rel="noreferrer noopener">用“等待-通知”机制优化循环等待</a><span class="ez-toc-section-end"></span> {.wp-block-heading}

当线程要求的条件满足时，用Java 对象的 notify() 和 notifyAll() 方法通知这个等待的线程。下面这个图里大致描述了这个过程，当条件满足时调用 notify()，会通知等待队列（互斥锁的等待队列）中的线程，告诉它**条件曾经满足过**。

<img decoding="async" src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0713b6722a664b3caa0997a78d3e1285~tplv-k3u1fbpfcp-watermark.awebp?" alt="image.png" /> 需要注意的是：因为**notify() 只能保证在通知时间点，条件是满足的**。而被通知线程的**执行时间点和通知的时间点**基本上不会重合，所以当线程执行的时候，很可能条件已经不满足了（因为可能有其他线程插队）。

<pre class="wp-block-code"><code lang="java" class="language-java line-numbers">  while(条件不满足) {
    wait();
  }</code></pre>

利用上面这种范式可以解决上面提到的**条件曾经满足**过这个问题。因为当 wait() 返回时，有可能条件已经发生变化了，曾经条件满足，但是现在已经不满足了，所以要重新检验条件是否满足。

<pre class="wp-block-code"><code lang="java" class="language-java line-numbers">// 此示例代码若参考课程中的背景介绍会更容易明白
class Allocator {
  private List&lt;Object&gt; als;
  // 一次性申请所有资源
  synchronized void apply(
    Object from, Object to){
    // 经典写法
    while(als.contains(from) ||
         als.contains(to)){
      try{
        wait();
      }catch(Exception e){
      }   
    } 
    als.add(from);
    als.add(to);  
  }
  // 归还资源
  synchronized void free(
    Object from, Object to){
    als.remove(from);
    als.remove(to);
    notifyAll();
  }
}</code></pre>

### <span class="ez-toc-section" id="%E9%94%81%E7%9A%84%E8%8E%B7%E5%8F%96%E4%B8%8E%E9%87%8A%E6%94%BE"></span><a href="https://link.juejin.cn?target=https%3A%2F%2Ftime.geekbang.org%2Fcolumn%2Farticle%2F87779" target="_blank" rel="noreferrer noopener">锁的获取与释放</a><span class="ez-toc-section-end"></span> {.wp-block-heading}

<pre class="wp-block-code"><code lang="java" class="language-java line-numbers">  lock.lock();
  try {
  } finally {
    // 保证锁能释放
    lock.unlock();
  }</code></pre>

Java SDK 里面 Lock 的使用，有一个经典的范例（如上），就是try{}finally{}，需要重点关注的是在 finally 里面释放锁。

<pre class="wp-block-code"><code lang="java" class="language-java line-numbers">// 此示例代码若参考课程中的背景介绍会更容易明白
class X {
  private final Lock rtl =
  new ReentrantLock();
  int value;
  public void addOne() {
    // 获取锁
    rtl.lock();  
    try {
      value+=1;
    } finally {
      // 保证锁能释放
      rtl.unlock();
    }
  }
}</code></pre>

### <span class="ez-toc-section" id="%E6%97%A0%E9%94%81%E5%B7%A5%E5%85%B7%E7%B1%BB"></span><a href="https://link.juejin.cn?target=https%3A%2F%2Ftime.geekbang.org%2Fcolumn%2Farticle%2F90515" target="_blank" rel="noreferrer noopener">无锁工具类</a><span class="ez-toc-section-end"></span> {.wp-block-heading}

<pre class="wp-block-code"><code lang="java" class="language-java line-numbers">// 这段抽象后的代码片段，它在很多无锁程序中经常出现
do {
  // 获取当前值
  oldV = xxxx；
  // 根据当前值计算新值
  newV = ...oldV...
}while(!compareAndSet(oldV,newV);
复制代码</code></pre>

在 Java 1.8 版本中，AtomicLong的getAndIncrement() 方法会转调 unsafe.getAndAddLong() 方法。查看以下JDK源码可以知道，这里通过`o`和`offset`两个参数可以唯一确定共享变量的内存地址。

<pre class="wp-block-code"><code lang="java" class="language-java line-numbers">public final long getAndAddLong(
  Object o, long offset, long delta){
  long v;
  do {
    // 读取内存中的值
    v = getLongVolatile(o, offset);
  } while (!compareAndSwapLong(
      o, offset, v, v + delta));
  return v;
}
//原子性地将变量更新为x
//条件是内存中的值等于expected
//更新成功则返回true
native boolean compareAndSwapLong(
  Object o, long offset, 
  long expected,
  long x);</code></pre>

### <span class="ez-toc-section" id="ThreadLocal_%E4%B8%8E%E5%86%85%E5%AD%98%E6%B3%84%E9%9C%B2"></span><a href="https://link.juejin.cn?target=https%3A%2F%2Ftime.geekbang.org%2Fcolumn%2Farticle%2F93745" target="_blank" rel="noreferrer noopener">ThreadLocal 与内存泄露</a><span class="ez-toc-section-end"></span> {.wp-block-heading}

在线程池中使用 ThreadLocal 为什么可能导致内存泄露呢？原因就出在线程池中线程的存活时间太长，往往都是和程序同生共死的，这就意味着 Thread 持有的 ThreadLocalMap 一直都不会被回收，再加上 ThreadLocalMap 中的 Entry 对 ThreadLocal 是弱引用（WeakReference），所以只要 ThreadLocal 结束了自己的生命周期是可以被回收掉的。但是 Entry 中的 Value 却是被 Entry 强引用的，所以即便 Value 的生命周期结束了，Value 也是无法被回收的，从而导致内存泄露。所以需要手动释放。

<pre class="wp-block-code"><code lang="java" class="language-java line-numbers">ExecutorService es;
ThreadLocal tl;
es.execute(()-&gt;{
  //ThreadLocal增加变量
  tl.set(obj);
  try {
    // 省略业务逻辑代码
  }finally {
    //手动清理ThreadLocal 
    tl.remove();
  }
});</code></pre>

### <span class="ez-toc-section" id="%E7%BB%88%E6%AD%A2%E7%BA%BF%E7%A8%8B"></span><a href="https://link.juejin.cn?target=https%3A%2F%2Ftime.geekbang.org%2Fcolumn%2Farticle%2F95847" target="_blank" rel="noreferrer noopener">终止线程</a><span class="ez-toc-section-end"></span> {.wp-block-heading}

按照两阶段终止模式，我们首先需要做的就是将线程 rptThread 状态转换到 RUNNABLE，做法很简单，只需要在调用 rptThread.interrupt() 就可以了。线程 rptThread 的状态转换到 RUNNABLE 之后，如何优雅地终止呢？下面的示例代码中，我们选择的标志位是线程的中断状态：Thread.currentThread().isInterrupted() ，需要注意的是，我们在捕获 Thread.sleep() 的中断异常之后，通过 Thread.currentThread().interrupt() 重新设置了线程的中断状态，因为 JVM 的异常处理会清除线程的中断状态。

在实际工作中应该尽量用如下代码。原因在于我们很可能在线程的 run() 方法中调用第三方类库提供的方法，而我们没有办法保证第三方类库正确处理了线程的中断异常，例如第三方类库在捕获到 Thread.sleep() 方法抛出的中断异常后，没有重新设置线程的中断状态，那么就会导致线程不能够正常终止。所以强烈建议你设置自己的线程终止标志位，例如在下面的代码中，使用 isTerminated 作为线程终止标志位，此时无论是否正确处理了线程的中断异常，都不会影响线程优雅地终止。

<pre class="wp-block-code"><code lang="java" class="language-java line-numbers">class Proxy {
  //线程终止标志位
  volatile boolean terminated = false;
  boolean started = false;
  //采集线程
  Thread rptThread;
  //启动采集功能
  synchronized void start(){
    //不允许同时启动多个采集线程
    if (started) {
      return;
    }
    started = true;
    terminated = false;
    rptThread = new Thread(()-&gt;{
      while (!terminated){
        //省略采集、回传实现
        report();
        //每隔两秒钟采集、回传一次数据
        try {
          Thread.sleep(2000);
        } catch (InterruptedException e){
          //重新设置线程中断状态
          Thread.currentThread().interrupt();
        }
      }
      //执行到此处说明线程马上终止
      started = false;
    });
    rptThread.start();
  }
  //终止采集功能
  synchronized void stop(){
    //设置中断标志位
    terminated = true;
    //中断线程rptThread
    rptThread.interrupt();
  }
}</code></pre>

PS：以上是我在掘金写的文章，所以图片带水印。