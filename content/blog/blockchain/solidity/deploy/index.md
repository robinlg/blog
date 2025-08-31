---
title: Difference new() with depoly() of Solidity for Smart Contract
date: 2018-12-06T09:01:10+00:00
categories:
  - 区块链
---
今天在做基于Ethereum的古董投资Dapp（编写完成后笔者会开源）单元测试时发觉了一个问题的存在，如题。我不确定new一个合约后返回的实例（一串十六进制的HASH）是否与deploy返回的HASH一样也是一个合约地址，在社区询问后无果，就自己在[remix][1]里写了一小段代码验证我的想法。希望能帮助读者解决这方面的困惑。

需要注意的是笔者在remix里的Environment是Injected Web3，使用MetaMask里的Kovan Test Network去完成测试。测试的Token可以在这里领取：<https://gitter.im/kovan-testnet/faucet>在gitter里发送你的钱包地址即可。下面进入正题：

<p style="text-align: center;">
  <img decoding="async" loading="lazy" class="aligncenter wp-image-336 size-full" src="http://roliu.work/wp-content/uploads/2018/12/1.png" alt="" width="1171" height="535" />图1
</p>

图1是我编写的一端简易的测试代码。我通过图右边的deploy直接将合约B部署上测试网络，在图左边的合约B代码中读者可以看见new了合约A的实例，那么我的猜想是否正确呢？我们将在测试网络的Ethereum区块链浏览器上看出结果。

<p style="text-align: center;">
  <img decoding="async" loading="lazy" class="aligncenter wp-image-337 size-full" src="http://roliu.work/wp-content/uploads/2018/12/2.png" alt="" width="1169" height="624" />图2
</p>

如图2，图左最上方的那条Transaction是我直接deploy后返回的结果（点进链接可查看详细内容，下面会详解）。图左下面三条是我new（点击图右下aInstance函数，详细代码看图1）实例后的结果。

<p style="text-align: center;">
  <img decoding="async" loading="lazy" class="aligncenter wp-image-338 size-full" src="http://roliu.work/wp-content/uploads/2018/12/3.png" alt="" width="1247" height="581" />图3
</p>

图3是点进deploy返回结果的链接显示的页面，图3下面的input Date是合约B的十六进制字节码，如图3所示创建红框里的合约并且返回其合约地址。对比图4、5、6，图4、5、6是new实例的详情。

<p style="text-align: center;">
  <img decoding="async" loading="lazy" class="aligncenter wp-image-339 size-full" src="http://roliu.work/wp-content/uploads/2018/12/4.png" alt="" width="1243" height="554" />图4
</p>

<p style="text-align: center;">
  <img decoding="async" loading="lazy" class="aligncenter wp-image-340 size-full" src="http://roliu.work/wp-content/uploads/2018/12/5.png" alt="" width="1244" height="562" />图5
</p>

<p style="text-align: center;">
  <img decoding="async" loading="lazy" class="aligncenter wp-image-341 size-full" src="http://roliu.work/wp-content/uploads/2018/12/6.png" alt="" width="1239" height="559" />图6
</p>

对比图3、4、5、6可以看出Transaction的对象（红框里的地址）都是同一串HASH，唯一不同的是图3是创建该合约，而其余三张是通过此合约再去创建另一合约（也就是创建新的合约地址）。所以图3没有图4、5、6最上面选项卡Internal Transaction。

<p style="text-align: center;">
  <img decoding="async" loading="lazy" class="aligncenter wp-image-342 size-full" src="http://roliu.work/wp-content/uploads/2018/12/7.png" alt="" width="1155" height="262" />图7
</p>

<p style="text-align: center;">
  <img decoding="async" loading="lazy" class="aligncenter wp-image-343 size-full" src="http://roliu.work/wp-content/uploads/2018/12/8.png" alt="" width="1149" height="257" />图8
</p>

<p style="text-align: center;">
  <img decoding="async" loading="lazy" class="aligncenter wp-image-344 size-full" src="http://roliu.work/wp-content/uploads/2018/12/9.png" alt="" width="1147" height="256" />图9
</p>

点进选项卡Internal Transaction，图7、8、9是new实例对象通过B合约创建的三个新的合约，图7、8、9红框中显示的是三个不同的合约地址。笔者点击图9红框中合约地址可见下图10的该合约的详情。

<p style="text-align: center;">
  <img decoding="async" loading="lazy" class="aligncenter wp-image-345 size-full" src="http://roliu.work/wp-content/uploads/2018/12/10.png" alt="" width="1158" height="559" />图10
</p>

<p style="text-align: center;">
  <img decoding="async" loading="lazy" class="aligncenter wp-image-346 size-full" src="http://roliu.work/wp-content/uploads/2018/12/11.png" alt="" width="1170" height="593" />图11
</p>

图10红框中的十六进制字节码与图11中合约A的字节码相同，点击进入图7、8的合约地址看到的字节码与图10、11的字节码也是相同的。即合约B每一次new都为创建合约A创建了新的合约地址，所以图1中Instance就是新的合约地址。与deploy效果一样返回的也是一个合约地址。

**若笔者文中存在错误之处，欢迎各位读者指正，共同学习。**

 [1]: http://remix.ethereum.org