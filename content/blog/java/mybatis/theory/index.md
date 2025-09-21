---
title: （二）Mybatis架构原理
date: 2020-08-15T04:50:18+00:00
categories:
  - Java
summary: 深入解析MyBatis三层架构设计，包括API结构、数据处理层、基础支撑层以及九大核心组件的工作原理和相互关系。
---

## <span class="ez-toc-section" id="%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1"></span>**架构设计**<span class="ez-toc-section-end"></span> {.wp-block-heading}

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2020/08/Mybatis架构设计-1024x897.png" alt="" class="wp-image-611" width="512" height="449" /><figcaption>图1 架构设计</figcaption></figure>
</div>

Mybatis的功能架构分为三层：  


### <span class="ez-toc-section" id="API%E7%BB%93%E6%9E%84"></span>**API结构**<span class="ez-toc-section-end"></span> {.wp-block-heading}

提供给外部使用的接口API，开发人员通过这些本地API来操纵数据库。接口层接收到调用请求就会调用数据处理层来完成具体的数据处理。

Mybatis和数据库的交互有两种方式：  
a. 使用传统的MyBatis提供的API；  
b. 使用Mapper代理的方式。

### <span class="ez-toc-section" id="%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%E5%B1%82"></span>**数据处理层**<span class="ez-toc-section-end"></span> {.wp-block-heading}

负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等。它主要的目的是根据调用的请求完成一次数据库操作。

### <span class="ez-toc-section" id="%E5%9F%BA%E7%A1%80%E6%94%AF%E6%92%91%E5%B1%82"></span>**基础支撑层**<span class="ez-toc-section-end"></span> {.wp-block-heading}

负责最基础的功能支撑、包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东西，将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑。

## <span class="ez-toc-section" id="%E4%B8%BB%E8%A6%81%E6%9E%84%E5%BB%BA%E5%8F%8A%E5%85%B6%E7%9B%B8%E4%BA%92%E5%85%B3%E7%B3%BB"></span>**主要构建及其相互关系**<span class="ez-toc-section-end"></span> {.wp-block-heading}<figure class="wp-block-table is-style-stripes">

<table>
  <tr>
    <td>
      构件
    </td>
    
    <td>
      描述
    </td>
  </tr>
  
  <tr>
    <td>
      SqlSession
    </td>
    
    <td>
      作为MyBatis工作的主要顶层API，<br />表示和数据库交互的会话，完成必要数据库层删改查。
    </td>
  </tr>
  
  <tr>
    <td>
      Executor
    </td>
    
    <td>
      MyBatis执行器，是MyBatis调度的核心，负责SQL语句<br />的生成和查询缓存的维护。
    </td>
  </tr>
  
  <tr>
    <td>
      StatementHandler
    </td>
    
    <td>
      封装了JDBC Statement操作，负责对JDBC Statement的<br />操作，如设置参数、将Statement结果集转换为List集<br />合。
    </td>
  </tr>
  
  <tr>
    <td>
      ParameterHandler
    </td>
    
    <td>
      负责对用户传递的参数转换成JDBC Statement所需要的<br />参数。
    </td>
  </tr>
  
  <tr>
    <td>
      ResultSetHandler
    </td>
    
    <td>
      负责将JDBC返回的ResultSet结果集对象转换成List类型<br />的集合。
    </td>
  </tr>
  
  <tr>
    <td>
      TypeHandler
    </td>
    
    <td>
      负责Java数据类型和JDBC数据类型之间的映射和转换。
    </td>
  </tr>
  
  <tr>
    <td>
      MappedStatement
    </td>
    
    <td>
      MappedStatement维护了一条<select/update/delete/<br />insert>结果的封装。
    </td>
  </tr>
  
  <tr>
    <td>
      SqlSource
    </td>
    
    <td>
      负责根据用户传递的parameterObject，动态地生成SQL<br />语句，将信息封装到BoundSql对象中，并返回。
    </td>
  </tr>
  
  <tr>
    <td>
      BoundSql
    </td>
    
    <td>
      表示动态生成的SQL语句以及相应的参数信息。
    </td>
  </tr>
</table></figure> 

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2020/08/Mybatis层次结构与JDBC-795x1024.png" alt="" class="wp-image-612" width="596" height="768" /><figcaption>图2 MyBatis层次结构</figcaption></figure>
</div>

## <span class="ez-toc-section" id="%E6%80%BB%E4%BD%93%E6%B5%81%E7%A8%8B"></span>**总体流程**<span class="ez-toc-section-end"></span> {.wp-block-heading}

### <span class="ez-toc-section" id="1%E5%8A%A0%E8%BD%BD%E9%85%8D%E7%BD%AE%E5%B9%B6%E5%88%9D%E5%A7%8B%E5%8C%96"></span>**1.加载配置并初始化**<span class="ez-toc-section-end"></span> {.wp-block-heading}

**触发条件：**加载配置文件  
配置来源于两处，一是配置文件（主配置文件conf.xml, mapper文件*.xml），一个是Java代码中的注释，将主配置文件内容解析封装到Configuration，将SQL的配置信息加载成为一个mappedStatement对象，储存在内存中

### <span class="ez-toc-section" id="2%E6%8E%A5%E6%94%B6%E8%B0%83%E7%94%A8%E8%AF%B7%E6%B1%82"></span>**2.接收调用请求**<span class="ez-toc-section-end"></span> {.wp-block-heading}

**触发条件：**调用Mybatis提供的API  
**传入参数：**SQL的ID和参数对象  
**处理过程：**将请求传递给下层的请求处理层进行处理。

### <span class="ez-toc-section" id="3%E5%A4%84%E7%90%86%E6%93%8D%E4%BD%9C%E8%AF%B7%E6%B1%82"></span>**3.处理操作请求**<span class="ez-toc-section-end"></span> {.wp-block-heading}

**触发条件：**API接口层传递请求过来  
**传入参数：**SQL的参数对象  
**处理过程：**  
a.根据SQL的ID查找对应的MappedStatement对象；  
b.根据传入的参数对象解析MappedStatement对象，得到最终要执行的SQL和执行传入的参数；  
c.获取数据库连接，根据最终得到的SQL语句和传入的参数到数据库执行，并得到执行结果；  
d.根据MappedStatement对象中的结果映射配置对得到的执行结果进行转换处理，并得到最终的处理结果；  
e.释放连接资源。

### <span class="ez-toc-section" id="4%E8%BF%94%E5%9B%9E%E5%A4%84%E7%90%86%E7%BB%93%E6%9E%9C"></span>**4.返回处理结果**<span class="ez-toc-section-end"></span> {.wp-block-heading}

返回最终的处理结果。