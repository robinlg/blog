---
title: The thinking of project about FISCO BCOS
date: 2019-12-30T02:24:56+00:00
categories:
  - 区块链
summary: 分享FISCO BCOS项目的深度思考，探讨联盟链技术架构、应用场景、开发实践以及区块链技术在企业级应用中的价值。
---

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="https://raw.githubusercontent.com/FISCO-BCOS/FISCO-BCOS/master/docs/images/FISCO_BCOS_Logo.svg?sanitize=true" alt="" /></figure>
</div>

<p class="has-regular-font-size">
  此篇博文以笔者参与的项目为背景进行编写，内容包括但不限于联盟链（ FISCO-BCOS ）在对该项目的业务与其技术实现中发挥的作用。在本文中笔者会多次引用到与业务或FISCO-BCOS的相关名词，若有疑问可查看本文引用的文章链接进行查看。以下内容为个人看法及观点，可能出现因考虑不足或经验缺乏导致的错误，若读者朋友发现任何问题及有任何疑问，还请不吝指教，我们共同探讨与进步，谢谢。
</p>

  * FISCO-BCOS能解决的实际问题
  * 如何发挥区块链的优势，在链上编写自己的业务逻辑
  * 使用SDK，让业务系统与区块链进行交互

在此项目前笔者更多的会关注公链，具体来说就是Ethereum，也着手写过一些Solidity智能合约，了解一些底层的技术实现。对联盟链的认知与“玩法”也只停留在听过像Fabric，R3等项目名词而已。那么通过这次机会熟悉了另一联盟链——FISCO-BCOS，关于其更多介绍可在文章引用链接部分查看。

## <span class="ez-toc-section" id="%E9%A1%B9%E7%9B%AE%E9%9D%A2%E4%B8%B4%E7%9A%84%E9%97%AE%E9%A2%98%E5%92%8C%E5%85%B6%E8%A7%A3%E5%86%B3%E4%B9%8B%E9%81%93"></span>**项目面临的问题和其解决之道**<span class="ez-toc-section-end"></span> {.wp-block-heading}

首先简单介绍项目（下面简称项目A），以便为下文做铺垫。项目A主要解决在无第三方机构储存个人明文资料的情况下，对这些个人资料做验证，保证其真实性以及没有被篡改。

众所周知，在中国大陆有许多类似学信网这样的网站由政府持有，作为储存个人资料和提供查询服务的第三方服务机构，这样的第三方服务机构储存了大量个人信息资料的网站。在笔者所参与的项目，其落地的区域隐私保护条例相对严格，在一些情况下，连政府机构也不允许随意流转与储存个人资料，那如何解决资料接收方查实资料真伪的问题呢？

FISCO-BCOS提供了这样的一个解决方案，在资料的拥有方与资料的查证方之间搭建起一个区块链网络，各方拥有各自的区块链节点，链上存储资料credential。资料的拥有方通过节点发布资料credential，资料查证方通过节点使用链下接收到的明文资料向区块链查询真伪，比对credential后即可知道查证方收到的资料是否与资料拥有方拥有的资料一致。  


在这个过程中，不存在有三方机构对资料的明文进行存储，也能确保资料的真实性。下面笔者针对项目A所涉及的业务，结合 FISCO-BCOS 提供的开源方案（1.提供链底层；2.提供一套智能合约作为链上的业务逻辑控制）进行分析与介绍。

在这里要注意的是，在链下传输证书明文时需注意安全问题，针对该项目的安全实施方式在此我就不详细展开，读者可以根据自己业务流程建立相应的安全机制。

## <span class="ez-toc-section" id="%E9%93%BE%E4%B8%8A%E7%9A%84%E4%B8%9A%E5%8A%A1%E9%80%BB%E8%BE%91%E2%80%94%E2%80%94%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6"></span>**链上的业务逻辑——智能合约**<span class="ez-toc-section-end"></span> {.wp-block-heading}

本文是以区块链应用层面展开描述，至于FISCO-BCOS提供的区块链底层支持细节，可以在**[参考资料[1]][1]**FISCO-BCOS查看。

笔者会以大笔篇幅来详述这一章节，以笔者角度来解析相关代码。这一套智能合约设计流程和相关描述可查看**[参考资料[2]][2]**。在阅读本章节前还应了解链上各机构扮演的角色及其作用**[参考资料[3]][3]**。

笔者会结合这些资料与读者一同解析代码，如果有错误的地方，欢迎在评论区提出。另， 读者在阅读此章节若具备智能合约相关知识，阅读起来会相对轻松。 FISCO-BCOS 与Ethereum的智能合约都使用Solidity编写，由于Solidity也是由于Ethereum才出现在大众的视野里，所以本文多处会提到Ethereum。 

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/sol.list_.png" alt="" class="wp-image-471" /><figcaption>图1 WeIdentity的智能合约列表</figcaption></figure>
</div>

请忽略图1列表中的红色下划波浪线，笔者先从权限控制功能的智能合约开始，再解析业务相关的智能合约部分。

### <span class="ez-toc-section" id="%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6%E9%83%A8%E5%88%86"></span>**权限控制部分**<span class="ez-toc-section-end"></span> {.wp-block-heading}



#### <span class="ez-toc-section" id="RoleControllersol"></span>**RoleController.sol**<span class="ez-toc-section-end"></span> {.wp-block-heading}

笔者从RoleController.sol开始解析，该合约保存了WeID相应的角色，并可从各角色中移除、添加WeID。下文如未特指某WeID表示为个人，WeID均默认表示为链上机构。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/Role_1.png" alt="" class="wp-image-473" /><figcaption>图2 定义的常量和变量</figcaption></figure>
</div>

图2定义了合约的错误码，角色映射的常量，操作映射的常量以及三个map结构。至于常量映射的关系从图中可以很直接地看出，不过多描述。三个map结构的key属性都为address（ 以太坊中的地址的长度为20字节，一字节等于8位，一共160位，所以address其实亦可以用uint160来声明 ），value的属性为boolean。

这些map可理解为角色，不同的map代表不同种的角色，一个角色包含许多机构（一对应一个map里有多个键值对）。由于区块链上的数据无法删除，意味着如果这些map里一旦添加了某机构，将无法将其删除，于是只能用true或false表示该机构在是否 某种角色中被删除。

  * authorityIssuerRoleBearer：表示某机构是否在“权威发布机构”这一角色中被移除（用ture，false表示）；
  * committeeMemberRoleBearer：表示某机构是否在“委员会成员”这一角色中被移除 （用ture，false表示）；
  * adminRoleBearer：表示某机构是否在“管理员”这一角色中被移除 （用ture，false表示）。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/Role_2.png" alt="" class="wp-image-475" /><figcaption>图3 合约的构造函数</figcaption></figure>
</div>

与.java类似，智能合约也有其构造函数，与合约同名。图3中的msg.sender表示调用该合约的WeID。注：在Ethereum中 msg.sender表示消息发送者的钱包地址或某合约地址，在FISCO-BCOS 表示 调用 者的WeID 或某合约地址 。

图3这段初始化函数代码：各角色数组记录了对应的机构的WeID（key值），以及其是否存在（value值，初始化为true）

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/Role_3-1.png" alt="" class="wp-image-478" /><figcaption>图4 查询各机构是否存在</figcaption></figure>
</div>

图4表示查询某种操作下，有无某机构。换句话说就是，查询某机构有无某种操作的权限。调用者应根据映射关系，使用uint值（opration）传入该合约函数里，根据不同operation查询不同的角色map。如果该map里存在查询机构的WeID（address）则返回true，否则返回false。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/Role_4.png" alt="" class="wp-image-480" /><figcaption>图5 增加角色map里的角色</figcaption></figure>
</div>

图5函数表示为增加角色， 调用者应根据映射关系，使用uint值（role）传入该合约函数里。合约根据传入的role区分角色，再将传入的address作为key保存在相应的角色map里，并设置value值为true，表示存在。注：图5的tx.orgin与图3的msg.sender虽然很都表示地址，但两者有不同之处，详细区别请读者自行查阅资料

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/Role_5.png" alt="" class="wp-image-481" /><figcaption>图6 移除角色map里的角色</figcaption></figure>
</div>

相应的，该函数为移除角色map里的角色。将某机构的value值设置为false，表示不存在（已移除）。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/Role_6.png" alt="" class="wp-image-482" /><figcaption>图7 查询机构角色是否存在</figcaption></figure>
</div>

图6函数表示，查询某角色（role）下是否存在某机构（address）。

与传统技术架构相对应的，智能合约做设计也应该分为逻辑控制层和数据持久层。由于在区块上的数据是通过智能合约地址索引而来，若业务逻辑需要更换，只需要将新的逻辑控制合约引入原来数据持久层的合约地址即可。这样的设计避免了由于数据和逻辑捆绑同一个合约，导致更换逻辑合约后原来的数据无法使用的问题。该项目逻辑控制和数据持久合约对应关系列表如下：

  * AuthorityIssuerController.sol &#8212; AuthorityIssuerData.sol
  * CommitteeMemberController.sol &#8212; CommitteeMemberData.sol
  * SpecificIssuerController.sol &#8212; SpecificIssuerData.sol
  * CptController.sol &#8212; CptData.sol
  * 

#### <span class="ez-toc-section" id="AuthorityIssuerDatasol"></span>**AuthorityIssuerData.sol**<span class="ez-toc-section-end"></span> {.wp-block-heading}

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/issuerData_1-1.png" alt="" class="wp-image-491" /><figcaption>图8 AuthorityIssuerData.sol</figcaption></figure>
</div>

图7定义了一些错误码，和AuthorityIssuer的结构体来表示一个权威发布机构实体。attribBytes32[0]表示机构名，attribInt[0]表示创建时间戳，accValue表示授权方累积判定值（ **[参考资料[4]][4]**/ **[参考资料[5]][5]**）。

定义了属性名为authorityIssuerMap的map，key的类型为address，value类型为 AuthorityIssuer 结构体； 属性名为 authorityIssuerArray的address数组；属性名为uniqueNameMap的map，key类型为bytes32，value类型为boolean。

图7最上方和RoleController.sol不同的是该合约从最开始导入了其它合约：“RoleController.sol”。图7这行代码表示：

<pre class="wp-block-preformatted">RoleController private roleController;</pre>

将导入的其它合约定义为该合约的对象，RoleController这个类名和其合约的构造函数相同。 

如图7下方， 传入RoleController的合约地址（ RoleController 合约需要在 AuthorityIssuerData 合约部署之前部署完毕，区块链会返回部署好的合约的地址），然后构造函数通过合约地址和图7中 RoleController 类名，将roleController初始化。  


<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/issuerDate_2.png" alt="" class="wp-image-494" /><figcaption>图9 查询权威发布机构是否存在</figcaption></figure>
</div>

图9注释已经解释清楚AuthorityIssuerData.sol与RoleController.sol保存的发布机构WwID不是同一份。前面也已经提过逻辑合约和数据合约存储的数据应该分开，这里也是同一道理。RoleController.sol可能在重新部署或者升级合约时，新的合约就没有发布机构的WeID。图9函数的功能就是查询权威发布机构是否存在。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/issuerDate_3.png" alt="" class="wp-image-495" /><figcaption>图10 查询权威机构名</figcaption></figure>
</div>

如图10，uniqueNameMap是保存权威发布机构唯一名map（key：机构名，value：boolean）。该函数返回boolean值

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/issuerData_4.png" alt="" class="wp-image-496" /><figcaption>图11 增加权威发布机构</figcaption></figure>
</div>

如图11，先使用图9方法查询发布机构是否存在，若不存在则继续执行。再使用图10方法查询发布机构名字是否存在，若不存在则继续执行。查询执行此方法的WeID是否有增加权威发布机构的权限，若有权限则继续。

后面的步骤分别是：在合约Controller中将addr（WeID）增加为权威发布机构；将addr &#8211; 结构体authorityIssuer，这个键值对放入authorityIssuerMap；将addr（WeID）push入authorityIssuerArray数组；将结构体authorityIssuer里发布机构的名字作为key放入uniqueNameMap，value为true。注：authorityIssuer前有memory标志符，其作用请读者自行查阅。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/issuerData_6.png" alt="" class="wp-image-497" /><figcaption>图12 删除权威发布机构</figcaption></figure>
</div>

如图12，流程如下：先查询该WeID是否存在，若存在则继续执行；查询执行此方法的WeID是否有删除权威发布机构的权限，若有则继续执行；调用roleController里removeRole方法删除该WeID的权限发布机构角色；delete为用于释放空间，更多说明请读者自行查阅资料；后面的代码的意义是将authorityIssuerArray数组最后一个addr（WeID）替换需要删除的（addr）WeID，并将数组最后一个元素的空间释放，这样就缩短了数组长度，因为区块链的空间是昂贵的（占用某一个节点的空间等于占用所有节点的空间）。

注：细心的读者可能会发现图12中index定义在里for循环里面，在后面index在循环外也被调用了。这里与Java不一样，Java语法不允许for循环里定义的字段在在循环外使用。  


#### <span class="ez-toc-section" id="AuthorityIssuerControllersol"></span>**AuthorityIssuerController.sol**<span class="ez-toc-section-end"></span> {.wp-block-heading}

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/issuer_1.png" alt="" class="wp-image-484" /><figcaption>图7 权威资料发布机构逻辑控制合约</figcaption></figure>
</div>

图7最上方和RoleController.sol不同的是该合约从最开始导入了其它两个合约：“AuthorityIssuerDatas.sol”和“RoleController.sol”。图7最下面两行代码表示，将导入的其它两个合约的定义为该合约的对象，AuthorityIssuerData和RoleController这两个类名和其合约的构造函数相同。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/issuer_2.png" alt="" class="wp-image-485" /><figcaption>图8 常量和事件</figcaption></figure>
</div>

图8定义了三个常量以及一个事件，事件在该合约里被调用。在Ethereum的区块结构里，有一块专用于存放event记录信息的空间，图8的operation/retCode/addr将会被存在此空间内。关于event的资料请读者自行查询。

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2019/06/issuer_3.png" alt="" class="wp-image-486" /><figcaption>图9 构造函数</figcaption></figure>
</div>

图9，传入AuthorityIssuerData.sol和RoleController.sol的合约地址（这两个合约需要在这合约部署之前部署完毕，部署合约区块链会返回部署好的合约地址），然后构造函数通过合约地址和图7中的类名，将authorityIssuerData和roleController初始化。

再次编辑这篇文章已经过了大半年了，想了一下还是把这篇文章推出去吧，也当挥洒当初的一腔热血。更多的代码和产品详情可通过文章下方的链接查阅，这篇文章笔者也不再另花时间往下写了。 WeIdentity 智能合约设计之道是值得借鉴与学习的，将处理逻辑与存储数据的合约区分开，以便于将旧数据接回新的逻辑合约（由于区块链数据不可删除等原因）。

从上次编辑文章至今FISCO BCOS推出了更多基于链所定制的一系列产品，有兴趣的读者可以通过链接查看。

<p class="has-vivid-red-color has-text-color">
  文章发布后不久，Weidentity已经进行大版本升级，升级后的合约设计已经是另一种实现思路，此文已再不适用新版本。
</p>

参考资料：

  * [1]FISCO-BCOS. WeBank. https://github.com/FISCO-BCOS
  * [2]WeIdentity.WeBank. https://github.com/WeBankFinTech/WeIdentity
  * [3]WeIdentity产品简介.WeBank. https://fintech.webank.com/weidentity
  * [4]WeIdentity规范文档.Authority Issuer注册.WeBank. https://weidentity.readthedocs.io/zh_CN/latest/docs/weidentity-spec.html#authority-issuer
  * [5]WeIdentity规范文档. Authority Issuer注册 .WeBank. https://weidentity.readthedocs.io/zh_CN/latest/docs/weidentity-spec.html#id20 
  * [6] WeIdentity智能合约代码.WeBank. https://github.com/WeBankFinTech/weid-contract

 [1]: https://github.com/FISCO-BCOS
 [2]: https://github.com/WeBankFinTech/WeIdentity
 [3]: https://fintech.webank.com/weidentity
 [4]: https://weidentity.readthedocs.io/zh_CN/latest/docs/weidentity-spec.html#authority-issuer
 [5]: https://weidentity.readthedocs.io/zh_CN/latest/docs/weidentity-spec.html#id20