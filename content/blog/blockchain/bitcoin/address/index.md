---
title: 区块链的1.0时代：比特币 – 第4章 高级密钥和地址
date: 2018-05-18T15:44:57+00:00
categories:
  - 区块链
series:
series_order: 5
---

### <span class="ez-toc-section" id="45_%E9%AB%98%E7%BA%A7%E5%AF%86%E9%92%A5%E5%92%8C%E5%9C%B0%E5%9D%80"></span>高级密钥和地址<span class="ez-toc-section-end"></span>

在以下部分中，我们将看到高级形式的密钥和地址，诸如加密私钥、脚本和多重签名地址，靓号地址，和纸钱包。

#### <span class="ez-toc-section" id="451_%E5%8A%A0%E5%AF%86%E7%A7%81%E9%92%A5%EF%BC%88BIP0038%EF%BC%89"></span>加密私钥（BIP0038）<span class="ez-toc-section-end"></span>

私钥必须保密。私钥的机密性需求事实情况是，在实践中相当难以实现，因为该需求与同样重要的安全对象可用性相互矛盾。当你需要为了避免私钥丢失而存储备份时，会发现维护私钥私密性是一件相当困难的事情。通过密码加密内有私钥的钱包可能要安全一点，但那个钱包也需要备份。有时，例如用户因为要升级或重装钱包软件，而需要把密钥从一个钱包转移到另一个。私钥备份也可能需要存储在纸张上（参见“4.5.4 纸钱包”一节）或者外部存储介质里，比如U盘。但如果一旦备份文件失窃或丢失呢？这些矛盾的安全目标推进了便携、方便、可以被众多不同钱包和比特币客户端理解的加密私钥标准BIP0038的出台。

BIP0038提出了一个通用标准，使用一个口令加密私钥并使用Base58Check对加密的私钥进行编码，这样加密的私钥就可以安全地保存在备份介质里，安全地在钱包间传输，保持密钥在任何可能被暴露情况下的安全性。这个加密标准使用了AES，这个标准由NIST建立，并广泛应用于商业和军事应用的数据加密。

BIP0038加密方案是：输入一个比特币私钥，通常使用WIF编码过，base58chek字符串的前缀“5”。此外BIP0038加密方案需要一个长密码作为口令，通常由多个单词或一段复杂的数字字母字符串组成。BIP0038加密方案的结果是一个由base58check编码过的加密私钥，前缀为6P。如果你看到一个6P开头的的密钥，这就意味着该密钥是被加密过，并需要一个口令来转换（解码）该密钥回到可被用在任何钱包WIT格式的私钥（前缀为5）。许多钱包APP现在能够识别BIP0038加密过的私钥，会要求用户提供口令解码并导入密钥。第三方APP，诸如非常好用基于浏览器的Bit Address，可以被用来解码BIP00038的密钥。

最通常使用BIP0038加密的密钥用例是纸钱包——一张纸张上备份私钥。只要用户选择了强口令，使用BIP0038加密的私钥纸钱包就无比的安全，是一种很棒的线下比特币存储途径（也被称作“冷库”）。

在bitaddress.org上测试表4-10中加密密钥，看看你如何得到输入口令的加密密钥。

表4-10 BIP0038加密私钥例子  
私钥（WIF） 5J3mBbAH58CpQ3Y5RNJpUKPE62SQ5tfcvU2JpbnkeyhfsYB1Jcn  
密码 MyTestPassphrase  
加密私钥（BIP0038） 6PRTHL6mWa48xSopbU1cKrVjpKbBZxcLRRCdctLJ3z5yxE87MobKoXdTsJ

#### <span class="ez-toc-section" id="452_P2SH_Pay-to-Script_Hash%E5%92%8C%E5%A4%9A%E9%87%8D%E7%AD%BE%E5%90%8D%E5%9C%B0%E5%9D%80"></span>P2SH (Pay-to-Script Hash)和多重签名地址<span class="ez-toc-section-end"></span>

正如我们所知，传统的比特币地址从数字1开头，来源于公钥，而公钥来源于私钥。虽然任何人都可以将比特币发送到一个1开头的地址，但比特币只能在通过相应的私钥签名和公钥哈希值后才能消费。

以数字3开头的比特币地址是P2SH地址，有时被错误的称谓多重签名或多重签名地址。他们指定比特币交易中受益人作为哈希的脚本，而不是公钥的所有者。这个特性在2012年1月由BIP0016引进，目前因为BIP0016提供了增加功能到地址本身的机会而被广泛的采纳。不同于发送资金到传统1开头的比特币地址的交易，也被称为P2PKH，资金被发送到3开头的地址时，不仅仅需要一个公钥的哈希值，同时也需要一个私钥签名作为所有者证明。在创建地址的时候，这些要求会被定义在脚本中，所有对地址的输入都会被这些要求阻隔。

一个P2SH地址从事务脚本中创建，它定义谁能消耗这个事务输出。（132页“P2SH（Pay-to-Script-Hash）”一节对此有详细的介绍）编码一个P2SH地址涉及使用一个在创建比特币地址用到过的双重哈希函数，并且只能应用在脚本而不是公钥：

script hash = RIPEMD160(SHA256(script))

脚本哈希的结果是由Base58Check编码前缀为5的版本、编码后得到开头为3的编码地址。一个P2SH地址例子是32M8ednmuyZ2zVbes4puqe44NZumgG92sM。

P2SH 不一定是多重签名的交易。虽然P2SH地址通常都是代表多重签名，但也可能是其他类型的交易脚本。

##### <span class="ez-toc-section" id="4521_%E5%A4%9A%E9%87%8D%E7%AD%BE%E5%90%8D%E5%9C%B0%E5%9D%80%E5%92%8CP2SH"></span>多重签名地址和P2SH<span class="ez-toc-section-end"></span>

目前，P2SH函数最常见的实现是用于多重签名地址脚本。顾名思义，底层脚本需要多个签名来证明所有权，此后才能消费资金。设计比特币多重签名特性是需要从总共N个密钥中需要M个签名（也被称为“阈值”），被称为M-的-N的多签名，其中M是等于或小于N。例如，第一章中提到的咖啡店主鲍勃使用多重签名地址需要1-2签名，一个是属于他的密钥和一个属于他同伴的密钥，以确保其中一方可以签署度过一个事务锁定输出到这个地址。这类似于传统的银行中的一个“联合账户”，其中任何一方配偶可以凭借单一签名消费。或Gopesh， Bob雇佣的网页设计师创立一个网站，可能为他的业务需要一个2-3的多签名地址，确保没有资金会被花费除非至少两个业务合作伙伴签署这笔交易。

我们将会在第五章节探索如何使用P2SH地址创建事务用来消费资金。

#### <span class="ez-toc-section" id="453_%E6%AF%94%E7%89%B9%E5%B8%81%E9%9D%93%E5%8F%B7%E5%9C%B0%E5%9D%80"></span>比特币靓号地址<span class="ez-toc-section-end"></span>

靓号地址包含了可读信息的有效比特币地址。例如，1LoveBPzzD72PUXLzCkYAtGFYmK5vYNR33就是包含了Base-58字母love的。靓号地址需要生成并通过数十亿的候选私钥测试，直到一个私钥能生成具有所需图案的比特币地址。虽然有一些优化过的靓号生成算法，该方法必须涉及随机上选择一个私钥，生成公钥，再生成比特币地址，并检查是否与所要的靓号图案相匹配，重复数十亿次，直到找到一个匹配。

一旦找到一个匹配所要图案的靓号地址，来自这个靓号地址的私钥可以和其他地址相同的方式被拥有者消费比特币。靓号地址不比其他地址具有更多安全性。它们依靠和其他地址相同的ECC和SHA。你无法比任何别的地址更容易的获得一个靓号图案开头的私钥。

在第一章中，我们介绍了Eugenia，一位在菲律宾工作的儿童慈善总监。我们假设Eugenia组织了一场比特币募捐活动，并希望使用靓号比特币地址来宣布这个募捐活动。Eugenia将会创造一个以1Kids开头的靓号地址来促进儿童慈善募捐的活动。让我们看看这个靓号地址如何被创建，这个靓号地址对Eugenia慈善募捐的安全性又意味着什么。

##### <span class="ez-toc-section" id="4531_%E7%94%9F%E6%88%90%E9%9D%93%E5%8F%B7%E5%9C%B0%E5%9D%80"></span>生成靓号地址<span class="ez-toc-section-end"></span>

我们必须认识到使用来自Base58字母表中简单符号来代表比特币地址是非常重要的。搜索“1kids”开头的图案我们会发现从1Kids11111111111111111111111111111到1Kidszzzzzzzzzzzzzzzzzzzzzzzzzzzzz的地址。这些以“1kid”开头的地址范围中大约有58的29次方地址。表4-11显示了这些有“1kids”前缀的地址。

表4-11 “1Kids”靓号的范围  
From 1Kids11111111111111111111111111111  
To 1Kidszzzzzzzzzzzzzzzzzzzzzzzzzzzzz

我们把“1Kids”这个前缀当作数字，我们可以看看比特币地址中这个前缀出现的频率。如果是一台普通性能的桌面电脑，没有任何特殊的硬件，可以每秒发现大约10万个密钥。

表4-12 靓号的出现的频率（1KidsCharity）以及生成所需时间  
长度 地址前缀 概率 平均生成时间  
1 1K 1/58 < 1毫秒  
2 1Ki 1/3,364 50毫秒  
3 1Kid 1/(195*103) < 2秒  
4 1Kids 1/(11*106) 1分钟  
5 1KidsC 1/(656*106) 1小时  
6 1KidsCh 1/(38*109) 2天  
7 1KidsCha 1/(2.2*1012) 3–4 月  
8 1KidsChar 1/(128*1012) 13–18年  
9 1KidsChari 1/(7*1015) 800年  
10 1KidsCharit 1/(400*1015) 46,000年  
11 1KidsCharity 1/(23*1018) 250万年

正如你所见，Eugenia将不会很快地创建出以“1KidsCharity”开头的靓号地址，即使她有数千台的电脑同时进行运算。每增加一个字符就会增加58倍的计算难度。超过七个字符的搜索模式通常需要专用的硬件才能被找出，譬如用户定制的具有多图形处理单元（GPU）的桌面级设备。那些通常是无法继续在比特币挖矿中盈利的钻机，被重新赋予了寻找靓号地址的任务。用GPU系统搜索靓号的速度比用通用CPU要快很多个量级。

另一种寻找靓号地址的方法是将工作外包给一个矿池里的靓号矿工们，如靓号矿池中的矿池。一个矿池是一种允许那些GPU硬件通过为他人寻找靓号地址来获得比特币的服务。对小额的账单，Eugenia可以外包搜索模式为7个字符靓号地址寻找工作，在几个小时内就可以得到结果，而不必用一个CPU搜索上几个月才得到结果。

生成一个靓号地址是一项通过蛮力的过程：尝试一个随机密钥，检查结果地址是否和所需的图案想匹配，重复这个过程直到成功找到为止。例4-8是个靓号矿工的例子，用C++程序来寻找靓号地址。这个例子运用到了我们在56页“其他替代客户端、资料库、工具包”一节介绍过的libbitcoin库。

例4-8 靓号挖掘程序

#include // The string we are searching forconst std::string search = &#8220;1kid&#8221;;// Generate a random secret key. A random 32 bytes.bc::ec\_secret random\_secret(std::default\_random\_engine& engine); // Extract the Bitcoin address from an EC secret.std::string bitcoin\_address(const bc::ec\_secret& secret);// Case insensitive comparison with the search string.bool match\_found(const std::string& address);int main() { std::random\_device random; std::default\_random\_engine engine(random()); // Loop continuously&#8230; while (true) { // Generate a random secret. bc::ec\_secret secret = random\_secret(engine); // Get the address. std::string address = bitcoin\_address(secret); // Does it match our search string? (1kid) if (match\_found(address)) { // Success! std::cout << &#8220;Found vanity address! &#8221; << address << std::endl; std::cout << &#8220;Secret: &#8221; << bc::encode\_hex(secret) << std::endl; return 0; } } // Should never reach here! return 0; }bc::ec\_secret random\_secret(std::default\_random\_engine& engine){ // Create new secret&#8230; bc::ec\_secret secret; // Iterate through every byte setting a random value&#8230; for (uint8\_t& byte: secret) byte = engine() % std::numeric\_limits::max(); // Return result. return secret;}std::string bitcoin\_address(const bc::ec\_secret& secret) { // Convert secret to pubkey&#8230; bc::ec\_point pubkey = bc::secret\_to\_public\_key(secret); // Finally create address. bc::payment\_address payaddr; bc::set\_public\_key(payaddr, pubkey); // Return encoded form. return payaddr.encoded(); }bool match\_found(const std::string& address) { auto addr\_it = address.begin(); // Loop through the search string comparing it to the lower case // character of the supplied address. for (auto it = search.begin(); it != search.end(); ++it, ++addr\_it) if (\*it != std::tolower(\*addr_it)) return false; // Reached end of search string, so address matches. return true; }

示例程序需要用C编译器链接libbitcoin库（此库需要提前装入该系统）进行编译。直接执行vanity-miner的可执行文件（不用参数，参见例4-9），它就会尝试碰撞以“1kid”开头的比特币地址。

例4-9 编译并运行vanity-miner程序示例

$ # Compile the code with g++$ g++ -o vanity-miner vanity-miner.cpp $(pkg-config &#8211;cflags &#8211;libs libbitcoin) $ # Run the example$ ./vanity-minerFound vanity address! 1KiDzkG4MxmovZryZRj8tK81oQRhbZ46YTSecret: 57cc268a05f83a23ac9d930bc8565bac4e277055f4794cbd1a39e5e71c038f3f$ # Run it again for a different result$ ./vanity-minerFound vanity address! 1Kidxr3wsmMzzouwXibKfwTYs5Pau8TUFnSecret: 7f65bbbbe6d8caae74a0c6a0d2d7b5c6663d71b60337299a1a2cf34c04b2a623# Use &#8220;time&#8221; to see how long it takes to find a result$ time ./vanity-minerFound vanity address! 1KidPWhKgGRQWD5PP5TAnGfDyfWp5yceXMSecret: 2a802e7a53d8aa237cd059377b616d2bfcfa4b0140bc85fa008f2d3d4b225349real 0m8.868suser 0m8.828ssys 0m0.035s

正如我们运行Unix命令time所测出的运行时间所示，示例代码要花几秒钟来找出匹配“kid”三个字符模板的结果。读者们可以在源代码中改变search这一搜索模板，看一看如果是四个字符或者五个字符的搜索模板需要花多久时间！

##### <span class="ez-toc-section" id="4532_%E9%9D%93%E5%8F%B7%E5%9C%B0%E5%9D%80%E5%AE%89%E5%85%A8%E6%80%A7"></span>靓号地址安全性<span class="ez-toc-section-end"></span>

靓号地址既可以增加、也可以削弱安全措施，它们着实是一把双刃剑。用于改善安全性时，一个独特的地址使对手难以使用他们自己的地址替代你的地址，以欺骗你的顾客支付他们的账单。不幸的是，靓号地址也可能使得任何人都能创建一个类似于随机地址的地址，甚至另一个靓号地址，从而欺骗你的客户。

Eugenia可以让捐款人捐款到她宣布的一个随机生成地址（例如：1J7mdg5rbQyUHENYdx39WVWK7fsLpEoXZy）。或者她可以生成一个以“1Kids”开头的靓号地址以显得更独特。

在这两种情况下，使用单一固定地址（而不是每比捐款用一个独立的动态地址）的风险之一是小偷有可能会黑进你的网站，用他自己的网址取代你的网址，从而将捐赠转移给自己。如果你在不同的地方公布了你的捐款地址，你的用户可以在付款之前直观地检查以确保这个地址跟在你的网站、邮件和传单上看到的地址是同一个。在随机地址1j7mdg5rbqyuhenydx39wvwk7fslpeoxzy的情况下，普通用户可能会只检查头几个字符“1j7mdg”，就认为地址匹配。使用靓号地址生成器，那些想通过替换类似地址来盗窃的人可以快速生成与前几个字符相匹配的地址，如表4-13所示。

表4-13 生成匹配某随机地址的多个靓号  
原版随机地址 1J7mdg5rbQyUHENYdx39WVWK7fsLpEoXZy  
4位字符匹配 1J7md1QqU4LpctBetHS2ZoyLV5d6dShhEy  
5位字符匹配 1J7mdgYqyNd4ya3UEcq31Q7sqRMXw2XZ6n  
6位字符匹配 1J7mdg5WxGENmwyJP9xuGhG5KRzu99BBCX

那靓号地址会不会增加安全性？如果Eugenia生成1Kids33q44erFfpeXrmDSz7zEqG2FesZEN的靓号地址，用户可能看到靓号图案的字母和一些字符在上面，例如在地址部分中注明了1Kids33。这样就会迫使攻击者生成至少6个字母相匹配的的靓号地址（比之前多2个字符），就要花费比Eugenia多3364倍的靓号图案。本质上，Eugenia付出的努力（或者靓号池付出的）迫使攻击者不得不生成更长的靓号图案。如果Eugenia花钱请矿池生成8个字符的靓号地址，攻击者将会被逼迫到10字符的境地，那将是个人电脑，甚至昂贵自定义靓号挖掘机或靓号池也无法生成。对Eugenia来说可承担的起支出，对攻击者来说则变成了无法承担支出，特别是如果欺诈的回报不足以支付生成靓号地址所需的费用。

#### <span class="ez-toc-section" id="454_%E7%BA%B8%E9%92%B1%E5%8C%85"></span>纸钱包<span class="ez-toc-section-end"></span>

纸钱包是打印在纸张上的比特币私钥。有时纸钱包为了方面起见也包括对应的比特币地址，但这并非是必要的。因为地址可以从私钥中导出。纸钱包是一个非常有效简历备份或者线下比特币存储方式，也是被称为“冷钱包”。作为备份机制，一个纸钱包可以提供安全性，以防在电脑硬盘损坏、失窃或意外删除的情况下造成密钥的的丢失。作为一个冷存储的机制，如果纸钱包密钥在线下生成并永久不在电脑系统中存储，他们在应对黑客攻击，键盘记录器，或其他在线电脑欺骗更有安全性。

纸钱包有许多不同的形状，大小，和外观设计，但非常基本的原则是一个密钥和一个地址打印在纸张上。表4-14展现了纸钱包最基本的形式。

表4-14 比特币纸钱包的私钥和公钥的打印形式  
公开地址 1424C2F4bC9JidNjjTUZCbUxv6Sa1Mt62x  
私钥（WIF） 5J3mBbAH58CpQ3Y5RNJpUKPE62SQ5tfcvU2JpbnkeyhfsYB1Jcn

通过使用工具，就可以很容易地生成纸钱包，譬如使用bitaddress.org网站上的客户端Javascript生成器。这个页面包含所有必要的代码，甚至在完全失去网络连接的情况下，也可以生成密钥和纸钱包。若要使用它，先将HTML页面保存在本地磁盘或外部U盘。从Internet网络断开，从浏览器中打开文件。更方便的，使用一个原始操作系统启动电脑，比如一个光盘启动的Linux系统。任何在脱机情况下使用这个工具所生成的密钥，都可以通过USB线在本地打印机上打印出来，从而制造了密钥只存在纸张上而从未存储在在线系统上的纸钱包。将这些纸钱包放置在防火容器内，发送比特币到对应的比特币地址上，从而实现了一个简单但非常有效的冷存储解决方案。图4-14展示了通过bitaddress.org 生成的纸钱包。

<p style="text-align: center;">
  <img decoding="async" loading="lazy" class="aligncenter wp-image-311 size-full" src="http://roliu.work/wp-content/uploads/2018/07/121910bydg2cmd0m0d6r28.png" alt="" width="486" height="260" />图4-14 通过bitaddress.org 生成的普通纸钱包
</p>

这个简单的纸钱包系统的不足之处是那些被打印下来的密钥容易被盗窃。一个能够获取接近这些纸币的小偷可以只需偷走纸币或者用把纸币上密钥拍摄下来，就能获得被这些密钥加密过的比特币的控制权。一个更复杂的纸钱包存储系统使用BIP0038加密的私钥。这些私钥被打印在纸钱包上被所有者记住的口令保护起来。没有口令，这些被加密过的密钥也是毫无用处的。但它们仍旧要比用口令保护的钱包级别要高，因为这些密钥从没有在线过，必须从物理上从保险箱或者其他物理安全存储中导出。图4-15展示了通过bitaddress.org 生成的加密纸钱包。

<img decoding="async" loading="lazy" class="aligncenter wp-image-312 size-full" src="http://roliu.work/wp-content/uploads/2018/07/121910wf91f7arzv1o3pyy.png" alt="" width="486" height="260" /> 

<p style="text-align: center;">
  图4-15 通过bitaddress.org 生成的加密纸钱包。密码是“test”。
</p>

虽然你可以多次存款到纸钱包中，但是你最好一次性提款，一次性提取里面所有的资金。因为如果你提取的金额少于其中的金额的话，会生成一个找零地址。并且，你所用的电脑可能被病毒感染，那么就有可能泄露私钥。一次性提款可以减少私钥泄露的风险，如果你所需的金额比较少，那么请把余额找零到另一个纸钱包中。

纸钱包有许多设计和大小，并有许多不同的特性。有些作为礼物送给他人，有季节性的主题，像圣诞节和新年主题。另外一些则是设计保存在银行金库或通过某种方式隐藏私钥的保险箱内，或者用不透明的刮刮贴，或者折叠和防篡改的铝箔胶粘密封。图4-16至图4-18展示了几个不同安全和备份功能的纸钱包的例子。

<img decoding="async" loading="lazy" class="aligncenter wp-image-313 size-full" src="http://roliu.work/wp-content/uploads/2018/07/121911tlii19zbsa9ilb1l.png" alt="" width="871" height="254" /> 

<p style="text-align: center;">
  图4-16 通过bitcoinpaperwallet.com生成的、私钥写在折叠皮瓣上的纸钱包
</p>

<img decoding="async" loading="lazy" class="aligncenter wp-image-315 size-full" src="http://roliu.work/wp-content/uploads/2018/07/121912tk4hsel4z5ew2gfl.png" alt="" width="400" height="203" /> 

<p style="text-align: center;">
  图4-17 通过bitcoinpaperwallet.com 生成的、私钥被密封住的纸钱包
</p>

其他设计有密钥和地址的额外副本，类似于票根形式的可以拆卸存根，让你可以存储多个副本以防火灾、洪水或其他自然灾害。

<img decoding="async" loading="lazy" class="aligncenter wp-image-316 size-full" src="http://roliu.work/wp-content/uploads/2018/07/121912y499vlofk91489k1.png" alt="" width="994" height="340" /> 

<p style="text-align: center;">
  图4-18 在备份“存根”上有多个私钥副本的纸钱包
</p>

[附录1 交易脚本的操作符、常量和符号][1]  
[附录2 比特币改进协议][2]  
[附录3 pycoin库、ku程序和tx交易程序][3]  
[附录4 sx工具下一些的命令][4]

本文原链接：[http://zhibimo.com/read/wang-miao/Mastering-Bitcoin/Chapter04.html][5]

 [1]: http://8btc.com/article-1795-1.html
 [2]: http://8btc.com/article-1796-1.html
 [3]: http://8btc.com/article-1797-1.html
 [4]: http://8btc.com/article-1798-1.html
 [5]: http://zhibimo.com/read/wang-miao/Mastering-Bitcoin/Chapter05.html