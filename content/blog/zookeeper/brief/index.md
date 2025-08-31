---
title: ZooKeeper简介与基本使用
date: 2021-11-30T00:00:00+00:00
categories:
  - Zookeeper
---
## **1. ZooKeeper简介** {.wp-block-heading}

举两个分布式系统协调的例子：

  * **通过网络进行信息共享**  
    好比：Leader在会上传达任务，组员通过听Leader命令或者看Leader邮件知道自己的任务。当任务分配有变化时，Leader会单独告诉组员，或者再次召开会议。信息通过人与人直接直接沟通，完成传递。

  * **通过共享存储**  
    好比：Leader按照约定的时间和路径，把任务分配表放到SVN上，组员每天去SVN上拉取最新的任务分配表，其中SVN就是共享存储。更好的做法是，当SVN文件版本更新时，组员都能第一时间得到消息，使自己手中的任务分配表永远是最新的，这种方式依赖于中央存储。过程如下图1所示：

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/wp-content/uploads/2020/12/共享存储-1-1024x911.png" alt="" class="wp-image-733" /><figcaption>图1</figcaption></figure>
</div>

#### **ZooKeeper如何解决分布式系统面临的问题** {.wp-block-heading}

ZooKeeper对分布式系统的协调，使用的是第二种「共享存储」的方式。  
实际上，通过ZooKeeper实现分布式协同的原理，和图1的例子是一样的。ZooKeeper就像是SVN，储存了任务的分配、完成情况等的共享信息。每个分布式应用系统就像是Leader和组员，订阅这些共享信息。当主系统（Leader）对某从系统（组员）的任务作出改变时，订阅了该任务的从系统会得到ZooKeeper的通知，取得自己最新的任务分配。参考下图2:

<div class="wp-block-image">
  <figure class="aligncenter"><img decoding="async" src="http://roliu.work/?attachment_id=734" alt="" /><figcaption>图2</figcaption></figure>
</div>

大多数分布式系统中出现的问题，都源于信息的共享。如果各应用系统间的信息不能及时共享和同步，那么就会在协作过程中产生各种问题。ZooKeeper解决协同问题的关键就是在于保证分布式系统信息的一致性。

**ZooKeeper的基本概念**