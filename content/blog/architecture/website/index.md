---
title: 网站应用架构的演进
date: 2021-03-21T07:58:19+00:00
categories:
  - 架构
summary: 详细介绍了网站应用架构从单体应用、垂直应用、SOA到微服务架构的演进过程，分析了每种架构的优缺点和适用场景。
---
 

随着互联网的发展，网站应用的规模不断扩大，网站的应用架构已经从最初的“单体应用架构”演进到现在的“微服务架构”。听说阿里在成立时，网站用的也是“单体应用架构”。BTW：我的毕业设计用的也是“单一应用架构” 😀

  * 单体应用架构
  * 垂直应用架构
  * SOA(Service-Oriented Architecture)应用架构
  * 微服务应用架构

现在还是有相当一部分系统用的是“单体应用架构”，我觉得是取决于用户的体量。简单、少人用的系统就无谓浪费太多资源去使用一个复杂的架构，毕竟越简单越少BUG。

毕业至今，自己曾经参与开发过的系统也仅有“单体应用架构”和“单体应用架构拓展版”。后面三种应用架构仅停留在自己搭建玩玩，真正上过生产的系统还没有。好奇心太强，始终还是想亲身经历挑战。

### <span class="ez-toc-section" id="%E5%8D%95%E4%BD%93%E5%BA%94%E7%94%A8%E6%9E%B6%E6%9E%84"></span>**单体应用架构**<span class="ez-toc-section-end"></span> {.wp-block-heading}

当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是关键。举个例子：本博客的架构就是“单体应用架构”。<figure class="wp-block-image is-resized">

<img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2021/03/单一应用架构-1.png" alt="" class="wp-image-794" width="615" height="428" /> <figcaption>单体应用架构  
</figcaption></figure> 

#### <span class="ez-toc-section" id="%E4%BC%98%E7%82%B9%EF%BC%9A"></span>**优点：**<span class="ez-toc-section-end"></span> {.wp-block-heading}

  * 项目可以快速开发完成，团队成员少，甚至一个人的时候都可快速迭代。
  * 架构简单：只需要借助IDE开发及调试即可。
  * 易于测试：只需要通过单元测试或者浏览器完成。
  * 易于部署：打包成单⼀可执⾏的jar或者打成war包放到容器内启动。

#### <span class="ez-toc-section" id="%E7%BC%BA%E7%82%B9%EF%BC%9A"></span>**缺点：**<span class="ez-toc-section-end"></span> {.wp-block-heading}

  * 随着功能不断迭代，单个项⽬过⼤，代码杂乱，耦合严重。开发团队逐渐壮⼤以后，沟通成本变高。
  * 新增业务困难：在已经非常杂乱的系统中增加新业务，维护旧功能是一件非常困难的事情，新人需要一段时间才能接手任务，学习成本变高。
  * 当核⼼业务与边缘业务混合一起，出现问题时会互相影响。如：⼀个临时活动的流量猛涨，使机器负载升⾼，可能会影响正常的核心服务。

业务量上涨之后，单体应⽤架构进⼀步丰富变化，⽐如应⽤集群部署、使⽤Nginx进⾏负载均衡、增加缓存服务器、增加⽂件服务器、数据库集群并做读写分离等，通过以上措施增强应对⾼并发的能⼒、应对⼀定的复杂业务场景，但依然属于单体应⽤架构。如下图：<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2021/03/单体应用架构扩展版-1024x646.png" alt="" class="wp-image-795" /> <figcaption>单体应用架构推展版</figcaption></figure> 

### <span class="ez-toc-section" id="%E5%9E%82%E7%9B%B4%E5%BA%94%E7%94%A8%E6%9E%B6%E6%9E%84"></span>**垂直应用架构**<span class="ez-toc-section-end"></span> {.wp-block-heading}

为了避免上⾯提到的那些问题，开始做模块的垂直划分。当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，提升效率的方法之一是将应用拆成互不相干的几个应用，以提升效率。此时，用于加速前端页面开发的Web框架(MVC)是关键。<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2021/03/垂直应用架构.png" alt="" class="wp-image-796" /> </figure> 

**优点：**

  * 系统拆分实现了流量分担，解决了并发问题。
  * 可以针对不同模块进⾏优化。
  * ⽅便⽔平扩展，负载均衡，容错率提⾼。
  * 系统间相互独⽴，互不影响，新的业务迭代时更加⾼效。

**缺点：**

  * 对于服务间的相互调⽤，若某个服务的端⼝或者ip地址发⽣改变，调⽤的系统需要⼿动更改。
  * 搭建集群后，实现负载均衡⽐较复杂，如：内⽹负载，在迁移机器时会影响调⽤⽅的路由，导致线上故障。
  * 服务之间调⽤⽅式不统⼀，基于 Http Client 、 Web Service ，接⼝协议不统⼀。
  * 服务监控不到位：除了端⼝、进程的监控外，没有调⽤的成功率、失败率、总耗时等等这些监控指标。

### <span class="ez-toc-section" id="SOA%E5%BA%94%E7%94%A8%E6%9E%B6%E6%9E%84"></span>**SOA应用架构**<span class="ez-toc-section-end"></span> {#mce_47.wp-block-heading}

当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的资源调度和治理中心(SOA)是关键。

阿⾥巴巴开源的Dubbo ，⼀款⾼性能、轻量级的开源Java RPC框架，它提供了三⼤核⼼能⼒：⾯向接⼝的远程⽅法调⽤，智能容错和负载均衡，以及服务⾃动注册和发现。

SOA (Service-Oriented Architecture)，即⾯向服务的架构。根据实际业务，把系统拆分成合适的、独⽴部署的模块，模块之间相互独⽴（通过Web Service/Dubbo等技术进⾏通信）。<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2021/03/SOA应用架构-1024x771.png" alt="" class="wp-image-797" /> <figcaption>SAO应用架构</figcaption></figure> 

**优点：**

  * 分布式、松耦合、扩展灵活、可重⽤。
  * 服务抽取粒度较⼤，服务调⽤⽅和提供⽅的接口耦合度较⾼

### <span class="ez-toc-section" id="%E5%BE%AE%E6%9C%8D%E5%8A%A1%E5%BA%94%E7%94%A8%E6%9E%B6%E6%9E%84"></span>**微服务应用架构**<span class="ez-toc-section-end"></span> {#mce_66.wp-block-heading}

微服务架构可以说是SOA架构的⼀种拓展，这种架构模式下它拆分粒度更⼩、服务更独⽴。把应⽤拆分成为⼀个个微⼩的服务，不同的服务可以使⽤不同的开发语⾔和存储，服务之间往往通过Restful等轻量级通信。微服务架构关键在于微⼩、独⽴、轻量级通信。

微服务是在 SOA 上做的升华粒度更加细致，微服务架构强调的⼀个重点是“业务需要彻底的组件化和服务化”。<figure class="wp-block-image is-resized">

<img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2021/03/微服务应用架构-1-1024x1013.png" alt="" class="wp-image-799" width="615" height="608" /> <figcaption>微服务应用架构</figcaption></figure> 

微服务架构和SOA架构相似但有不同，微服务架构和SOA架构很明显的⼀个区别就是服务拆分粒度的不同。

举个例子：在SOA应用架构中“参与活动”和“任务中心”都有订单相关的需求，在这两个模块中各维护了一套订单相关需求的代码。在微服务架构中，将服务拆分为更细粒度，拆分为订单基础服务，那么其他两个模块调用这个订单基础服务即可。

**优点：**

  * 微服务很⼩，便于特定业务功能的聚焦 。
  * 微服务很⼩，每个微服务都可以被⼀个⼩团队单独实施（开发、测试、部署上线、运维），团队合作⼀定程度解耦，便于实施敏捷开发。
  * 微服务很⼩，便于重⽤和模块之间的组装。
  * 微服务很独⽴，那么不同的微服务可以使⽤不同的语⾔开发，松耦合。
  * 微服务架构下，我们更容易引⼊新技术。
  * 微服务架构下，我们可以更好的实现DevOps开发运维⼀体化。

**优点：**

  * 微服务架构下，分布式复杂难以管理，当服务数量增加，管理将越加复杂。
  * 微服务架构下，分布式链路跟踪难等。



文章参考: [Dubbo 2.7背景][1]

 [1]: https://dubbo.apache.org/zh/docs/v2.7/user/preface/background/