---
title: （一）自定义持久层设计思路
date: 2020-07-05T03:30:07+00:00
categories:
  - Java
summary: 详细介绍自定义持久层框架的设计思路，包括配置文件解析、JavaBean容器、SqlSessionFactory工厂模式以及Executor执行器等核心组件设计。
---
笔者开始对Mybaits源码进行分析、学习，并记录下学习的历程，既对自己的成长有帮助，也对有可能看到本文的读者起到帮助。<figure class="wp-block-image">

<img decoding="async" src="http://roliu.work/wp-content/uploads/2020/07/自定义持久层设计思路-1024x422.png" alt="" class="wp-image-576" /> <figcaption>图1 自定义持久层设计思路</figcaption></figure> 

对直接使用JDBC链接数据库的读者或许知道，要操作数据库需要以下几部分（相关JDBC操作数据库代码可自行Google/Baidu）：

  1. 指定驱动类;
  2. 指定数据库链接地址、数据库登陆账号及密码;
  3. SQL语句;
  4. 预执行SQL语句（PrepareStatement）;
  5. 设置参数及执行；
  6. 封装返回结果集。

使用持久层框架可以在业务系统简化以上步骤的代码，将链接的创建、销毁；SQL的执行与结果封装等这些“模版类”代码交由持久层负责，程序员只需要关注业务代码（SQL）即可。其中1与2步骤交由业务系统指定，3-6由持久层框架本身处理。

<div id="ez-toc-container" class="ez-toc-v2_0_56_1 counter-hierarchy ez-toc-counter ez-toc-grey ez-toc-container-direction">
  <div class="ez-toc-title-container">
    <p class="ez-toc-title " >
      目录
    </p>
    
    <span class="ez-toc-title-toggle"><a href="#" class="ez-toc-pull-right ez-toc-btn ez-toc-btn-xs ez-toc-btn-default ez-toc-toggle" aria-label="Toggle Table of Content" role="button"><label for="item-6896f65950c5b" ><span class=""><span style="display:none;">Toggle</span><span class="ez-toc-icon-toggle-span"><svg style="fill: #999;color:#999" xmlns="http://www.w3.org/2000/svg" class="list-377408" width="20px" height="20px" viewBox="0 0 24 24" fill="none"><path d="M6 6H4v2h2V6zm14 0H8v2h12V6zM4 11h2v2H4v-2zm16 0H8v2h12v-2zM4 16h2v2H4v-2zm16 0H8v2h12v-2z" fill="currentColor"></path></svg><svg style="fill: #999;color:#999" class="arrow-unsorted-368013" xmlns="http://www.w3.org/2000/svg" width="10px" height="10px" viewBox="0 0 24 24" version="1.2" baseProfile="tiny"><path d="M18.2 9.3l-6.2-6.3-6.2 6.3c-.2.2-.3.4-.3.7s.1.5.3.7c.2.2.4.3.7.3h11c.3 0 .5-.1.7-.3.2-.2.3-.5.3-.7s-.1-.5-.3-.7zM5.8 14.7l6.2 6.3 6.2-6.3c.2-.2.3-.5.3-.7s-.1-.5-.3-.7c-.2-.2-.4-.3-.7-.3h-11c-.3 0-.5.1-.7.3-.2.2-.3.5-.3.7s.1.5.3.7z"/></svg></span></span></label><input aria-label="Toggle" aria-label="item-6896f65950c5b"  type="checkbox" id="item-6896f65950c5b" /></a></span>
  </div><nav>
  
  <ul class='ez-toc-list ez-toc-list-level-1 ' >
    <li class='ez-toc-page-1 ez-toc-heading-level-3'>
      <a class="ez-toc-link ez-toc-heading-1" href="http://43.133.160.241/?p=575/#%E9%A1%B9%E7%9B%AE%E5%B7%A5%E7%A8%8B%EF%BC%88%E4%B8%9A%E5%8A%A1%E7%B3%BB%E7%BB%9F%EF%BC%89" title="项目工程（业务系统）">项目工程（业务系统）</a><ul class='ez-toc-list-level-4'>
        <li class='ez-toc-heading-level-4'>
          <a class="ez-toc-link ez-toc-heading-2" href="http://43.133.160.241/?p=575/#1_%E6%95%B0%E6%8D%AE%E5%BA%93%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%EF%BC%88%E8%87%AA%E5%AE%9A%E4%B9%89%E4%B8%BA%EF%BC%9AsqlMapperConfigxml%EF%BC%89" title="1. 数据库配置信息（自定义为：sqlMapperConfig.xml）">1. 数据库配置信息（自定义为：sqlMapperConfig.xml）</a>
        </li>
      </ul>
    </li>
    
    <li class='ez-toc-page-1 ez-toc-heading-level-3'>
      <a class="ez-toc-link ez-toc-heading-3" href="http://43.133.160.241/?p=575/#%E8%87%AA%E5%AE%9A%E4%B9%89%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6%EF%BC%88Jar%E5%8C%85%EF%BC%89" title="自定义持久层框架（Jar包）">自定义持久层框架（Jar包）</a><ul class='ez-toc-list-level-4'>
        <li class='ez-toc-heading-level-4'>
          <a class="ez-toc-link ez-toc-heading-4" href="http://43.133.160.241/?p=575/#%E6%AD%A5%E9%AA%A41%EF%BC%9A%E5%8A%A0%E8%BD%BD%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6" title="步骤1：加载配置文件">步骤1：加载配置文件</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-4'>
          <a class="ez-toc-link ez-toc-heading-5" href="http://43.133.160.241/?p=575/#%E6%AD%A5%E9%AA%A42_%E5%88%9B%E5%BB%BA%E4%B8%A4%E4%B8%AAjavaBean" title="步骤2: 创建两个javaBean">步骤2: 创建两个javaBean</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-4'>
          <a class="ez-toc-link ez-toc-heading-6" href="http://43.133.160.241/?p=575/#%E6%AD%A5%E9%AA%A43_%E8%A7%A3%E6%9E%90%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6" title="步骤3: 解析配置文件">步骤3: 解析配置文件</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-4'>
          <a class="ez-toc-link ez-toc-heading-7" href="http://43.133.160.241/?p=575/#%E6%AD%A5%E9%AA%A44_%E5%88%9B%E5%BB%BASqlSessionFactory%E6%8E%A5%E5%8F%A3%E5%8F%8A%E5%AE%9E%E7%8E%B0%E7%B1%BBDefaultSqlSessionFactory" title="步骤4: 创建SqlSessionFactory接口及实现类DefaultSqlSessionFactory">步骤4: 创建SqlSessionFactory接口及实现类DefaultSqlSessionFactory</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-4'>
          <a class="ez-toc-link ez-toc-heading-8" href="http://43.133.160.241/?p=575/#%E6%AD%A5%E9%AA%A45_%E5%88%9B%E5%BB%BASqlSession%E6%8E%A5%E5%8F%A3%E5%8F%8A%E5%AE%9E%E7%8E%B0%E7%B1%BBDefaultSession" title="步骤5: 创建SqlSession接口及实现类DefaultSession">步骤5: 创建SqlSession接口及实现类DefaultSession</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-4'>
          <a class="ez-toc-link ez-toc-heading-9" href="http://43.133.160.241/?p=575/#%E6%AD%A5%E9%AA%A46_%E5%88%9B%E5%BB%BAExecutor%E6%8E%A5%E5%8F%A3%E5%8F%8A%E5%AE%9E%E7%8E%B0%E7%B1%BBSampleExecutor" title="步骤6: 创建Executor接口及实现类SampleExecutor">步骤6: 创建Executor接口及实现类SampleExecutor</a>
        </li>
      </ul>
    </li>
  </ul></nav>
</div>

### <span class="ez-toc-section" id="%E9%A1%B9%E7%9B%AE%E5%B7%A5%E7%A8%8B%EF%BC%88%E4%B8%9A%E5%8A%A1%E7%B3%BB%E7%BB%9F%EF%BC%89"></span>**项目工程（业务系统）**<span class="ez-toc-section-end"></span> {.wp-block-heading}

业务系统引入自定义持久层的Jar包，并提供Jar包所需要的两种配置信息：

#### <span class="ez-toc-section" id="1_%E6%95%B0%E6%8D%AE%E5%BA%93%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%EF%BC%88%E8%87%AA%E5%AE%9A%E4%B9%89%E4%B8%BA%EF%BC%9AsqlMapperConfigxml%EF%BC%89"></span>**1. 数据库配置信息（自定义为：sqlMapperConfig.xml）**<span class="ez-toc-section-end"></span> {.wp-block-heading}

  * 数据库驱动类，如：com.msyql.jdbc.Driver等；
  * 数据库地址及账号密码；
  * mapper.xml全路径信息（在此处定义mapper.xml全路径信息的好处是Jar包只需要配置sqlMapperConfig.xml的路径即可，方便扩展）。

**2. SQL配置信息（自定义为：mapper.xml）**

  * SQL语句
  * 参数类型
  * 返回值类型

### <span class="ez-toc-section" id="%E8%87%AA%E5%AE%9A%E4%B9%89%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6%EF%BC%88Jar%E5%8C%85%EF%BC%89"></span>**自定义持久层框架（Jar包）**<span class="ez-toc-section-end"></span> {.wp-block-heading}

持久层框架的本质就是对JDBC代码进行封装

#### <span class="ez-toc-section" id="%E6%AD%A5%E9%AA%A41%EF%BC%9A%E5%8A%A0%E8%BD%BD%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6"></span>**步骤1：加载配置文件**<span class="ez-toc-section-end"></span> {.wp-block-heading}

创建Resources类，根据配置文件的路径，加载配置文件成字节输入流，存放至内存中。  
类的方法：InputStream getResourceAsStram(String path)

#### <span class="ez-toc-section" id="%E6%AD%A5%E9%AA%A42_%E5%88%9B%E5%BB%BA%E4%B8%A4%E4%B8%AAjavaBean"></span>**步骤2: 创建两个javaBean**<span class="ez-toc-section-end"></span> {.wp-block-heading}

创建两个容器对象（Configuration和MappedStatement），存放配置文件解析出来的内容。  
Configuration（核心配置类）：存放sqlMapperConfig.xml解析出来的内容；  
MappedStatement（映射配置类）：存放mapper.xml解析出来的内容。

#### <span class="ez-toc-section" id="%E6%AD%A5%E9%AA%A43_%E8%A7%A3%E6%9E%90%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6"></span>**步骤3: 解析配置文件**<span class="ez-toc-section-end"></span> {.wp-block-heading}

创建SqlSessionFactoryBuilder类。  
类的方法：build(InputStream) {  
1. 使用dom4j解析配置文件，将解析出来的内容封装到容器对象中；  
2. 创建sqlSessionFactory对象，生产sqlSession（对话对象），此处设计模式工厂模式。  
}

#### <span class="ez-toc-section" id="%E6%AD%A5%E9%AA%A44_%E5%88%9B%E5%BB%BASqlSessionFactory%E6%8E%A5%E5%8F%A3%E5%8F%8A%E5%AE%9E%E7%8E%B0%E7%B1%BBDefaultSqlSessionFactory"></span>**步骤4: 创建SqlSessionFactory接口及实现类DefaultSqlSessionFactory**<span class="ez-toc-section-end"></span> {.wp-block-heading}

类的方法：openSession() {  
生产sqlSession  
}

#### <span class="ez-toc-section" id="%E6%AD%A5%E9%AA%A45_%E5%88%9B%E5%BB%BASqlSession%E6%8E%A5%E5%8F%A3%E5%8F%8A%E5%AE%9E%E7%8E%B0%E7%B1%BBDefaultSession"></span>**步骤5: 创建SqlSession接口及实现类DefaultSession**<span class="ez-toc-section-end"></span> {.wp-block-heading}

定义对数据库的curd操作：

  * selectList()
  * selectOne()
  * update()
  * delete()

#### <span class="ez-toc-section" id="%E6%AD%A5%E9%AA%A46_%E5%88%9B%E5%BB%BAExecutor%E6%8E%A5%E5%8F%A3%E5%8F%8A%E5%AE%9E%E7%8E%B0%E7%B1%BBSampleExecutor"></span>**步骤6: 创建Executor接口及实现类SampleExecutor**<span class="ez-toc-section-end"></span> {.wp-block-heading}

类的方法：query(Configuration configuration, MappedStatement mappedStatement, Object&#8230; params) {  
执行的就是JDBC代码。  
}

<p class="has-vivid-cyan-blue-color has-text-color has-regular-font-size">
  <br /><strong>代码实现：</strong>
</p>

  * [自定义持久层框架代码参考实现][1]
  * [自定义持久层框架使用端代码实现][2]

 [1]: https://github.com/RobinLG/IPersistence
 [2]: https://github.com/RobinLG/IPersistence_test