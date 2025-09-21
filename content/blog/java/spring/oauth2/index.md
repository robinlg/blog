---
title: OAuth2
date: 2021-04-05T07:37:31+00:00
categories:
  - Java
summary: 详细介绍OAuth2开放授权协议，包括授权码模式和密码式两种授权方式，以及使用Spring Cloud OAuth2构建微服务统一认证服务的完整实现。
---

今天是清明假期最后一天，跟以往一样下起了细雨。天气越舒适，人的思绪就好像会越乱。年轻人和长辈的想法总会有出入，早上家人提出的话题被我草草结束。虽表面上结束了这个话题，但其实关于这个话题的联想一直脑海萦绕。还是写一篇技术博文冷静一下吧，生活上事情不提太多，毕竟这里是技术频道。 😀

上个月我们小组接到一个新项目。其中登录模块允许用户使用第三方账户登录，以下简称“账户系统”。后来运维“账户系统”的公司给我们发了一份技术文档，我在文档看完后就知道该“账户系统”使用了现在业界比较流行的“OAuth2开放授权协议”来允许⽤户授权第三⽅应⽤访问他们存储在服务提供者里的信息。

恰好最近也在看OAuth2的资料，于是顺便结合新项目（以下简称“表单系统”）来谈论一下OAuth2。首先通过简图来介绍一下系统的结构和方案，如下图1：

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2021/04/系统结构简图.png" alt="" class="wp-image-807" width="391" height="297" /><figcaption>图1 系统结构简图</figcaption></figure>
</div>

上图中省略了数据库和其他组件，实现OAuth2协议的代码都已经在账户系统（资源服务器和认证服务器）完成。我们开发的表单系统提供必要的参数（client_id, secret&#8230;）即可获取账户系统返回的授权码，再用授权码换取token，用token换取用户信息并保存，最后表单系统给用户创建session。这样就完成了‘用户授权“表单系统”访问他们储存在账户系统里的信息’登录的流程。简单流程图如下图2:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2021/04/产学研第三方登陆流程图-1024x294.png" alt="" class="wp-image-810" width="512" height="147" /><figcaption>图2 登录流程</figcaption></figure>
</div>

OAuth2常用的两种颁发token授权方式：

  1. 授权码(authorization-code)
  2. 密码式(password)

上述的表单系统登录使用的是第一种授权码模式，也是OAuth2协议里最复杂的一种模式，授权码模式使⽤到了回调地址，微博、微信、QQ等第三⽅登录就是这种模式。

下面笔者会结合代码梳理常用方式中比较简单的密码式。代码使用Spring Cloud OAuth2构建微服务统⼀认证服务。

Spring Cloud OAuth2 是 Spring Cloud 体系对OAuth2协议的实现，可以⽤来做多个微服务的统⼀认证（验证身份合法性）授权（验证权限）。通过向OAuth2服务（统⼀认证授权服务）发送某个类型的grant\_type进⾏集中认证和授权，从⽽获得access\_token（访问令牌），⽽这个token是受其他微服务信任的。简单流程如下图3:

<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img decoding="async" loading="lazy" src="http://roliu.work/wp-content/uploads/2021/04/Spring-Cloud-OAuth2流程-1024x498.png" alt="" class="wp-image-811" width="512" height="249" /><figcaption>图3 微服务统一认证简单流程</figcaption></figure>
</div>

<div id="ez-toc-container" class="ez-toc-v2_0_56_1 counter-hierarchy ez-toc-counter ez-toc-grey ez-toc-container-direction">
  <div class="ez-toc-title-container">
    <p class="ez-toc-title " >
      目录
    </p>
    
    <span class="ez-toc-title-toggle"><a href="#" class="ez-toc-pull-right ez-toc-btn ez-toc-btn-xs ez-toc-btn-default ez-toc-toggle" aria-label="Toggle Table of Content" role="button"><label for="item-6896f659db738" ><span class=""><span style="display:none;">Toggle</span><span class="ez-toc-icon-toggle-span"><svg style="fill: #999;color:#999" xmlns="http://www.w3.org/2000/svg" class="list-377408" width="20px" height="20px" viewBox="0 0 24 24" fill="none"><path d="M6 6H4v2h2V6zm14 0H8v2h12V6zM4 11h2v2H4v-2zm16 0H8v2h12v-2zM4 16h2v2H4v-2zm16 0H8v2h12v-2z" fill="currentColor"></path></svg><svg style="fill: #999;color:#999" class="arrow-unsorted-368013" xmlns="http://www.w3.org/2000/svg" width="10px" height="10px" viewBox="0 0 24 24" version="1.2" baseProfile="tiny"><path d="M18.2 9.3l-6.2-6.3-6.2 6.3c-.2.2-.3.4-.3.7s.1.5.3.7c.2.2.4.3.7.3h11c.3 0 .5-.1.7-.3.2-.2.3-.5.3-.7s-.1-.5-.3-.7zM5.8 14.7l6.2 6.3 6.2-6.3c.2-.2.3-.5.3-.7s-.1-.5-.3-.7c-.2-.2-.4-.3-.7-.3h-11c-.3 0-.5.1-.7.3-.2.2-.3.5-.3.7s.1.5.3.7z"/></svg></span></span></label><input aria-label="Toggle" aria-label="item-6896f659db738"  type="checkbox" id="item-6896f659db738" /></a></span>
  </div><nav>
  
  <ul class='ez-toc-list ez-toc-list-level-1 ' >
    <li class='ez-toc-page-1 ez-toc-heading-level-3'>
      <a class="ez-toc-link ez-toc-heading-1" href="http://43.133.160.241/?p=805/#%E6%90%AD%E5%BB%BA%E8%AE%A4%E8%AF%81%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%88Authorization_Server%EF%BC%89" title="搭建认证服务器（Authorization Server）">搭建认证服务器（Authorization Server）</a><ul class='ez-toc-list-level-4'>
        <li class='ez-toc-heading-level-4'>
          <a class="ez-toc-link ez-toc-heading-2" href="http://43.133.160.241/?p=805/#%E6%96%B0%E5%BB%BA%E9%A1%B9%E2%BD%ACrobin-cloud-oauth-server-9999%EF%BC%8Cpomxml" title="新建项⽬robin-cloud-oauth-server-9999，pom.xml:">新建项⽬robin-cloud-oauth-server-9999，pom.xml:</a>
        </li>
        <li class='ez-toc-page-1 ez-toc-heading-level-4'>
          <a class="ez-toc-link ez-toc-heading-3" href="http://43.133.160.241/?p=805/#%E8%AE%A4%E8%AF%81%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%89%E5%85%A8%E9%85%8D%E7%BD%AE%E7%B1%BB" title="认证服务器安全配置类">认证服务器安全配置类</a>
        </li>
      </ul>
    </li>
    
    <li class='ez-toc-page-1 ez-toc-heading-level-3'>
      <a class="ez-toc-link ez-toc-heading-4" href="http://43.133.160.241/?p=805/#%E8%B5%84%E6%BA%90%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%88Resource_Server%EF%BC%89" title="资源服务器（Resource Server）">资源服务器（Resource Server）</a><ul class='ez-toc-list-level-4'>
        <li class='ez-toc-heading-level-4'>
          <a class="ez-toc-link ez-toc-heading-5" href="http://43.133.160.241/?p=805/#%E8%B5%84%E6%BA%90%E6%9C%8D%E5%8A%A1%E9%85%8D%E7%BD%AE%E7%B1%BB" title="资源服务配置类">资源服务配置类</a>
        </li>
      </ul>
    </li>
  </ul></nav>
</div>

### <span class="ez-toc-section" id="%E6%90%AD%E5%BB%BA%E8%AE%A4%E8%AF%81%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%88Authorization_Server%EF%BC%89"></span>**搭建认证服务器（Authorization Server）**<span class="ez-toc-section-end"></span> {.wp-block-heading}

认证服务器（Authorization Server），负责颁发token。注意：运行此服务前需要先运行Eureka服务。

#### <span class="ez-toc-section" id="%E6%96%B0%E5%BB%BA%E9%A1%B9%E2%BD%ACrobin-cloud-oauth-server-9999%EF%BC%8Cpomxml"></span>**新建项⽬robin-cloud-oauth-server-9999，pom.xml:**<span class="ez-toc-section-end"></span> {.wp-block-heading}

<pre class="wp-block-preformatted">&lt;?xml version="1.0" encoding="UTF-8"?&gt;<br />&lt;project xmlns="http://maven.apache.org/POM/4.0.0"<br />         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"<br />         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"&gt;<br />    &lt;parent&gt;<br />        &lt;artifactId&gt;robin-parent&lt;/artifactId&gt;<br />        &lt;groupId&gt;com.robin&lt;/groupId&gt;<br />        &lt;version&gt;1.0-SNAPSHOT&lt;/version&gt;<br />    &lt;/parent&gt;<br />    &lt;modelVersion&gt;4.0.0&lt;/modelVersion&gt;<br /><br />    &lt;groupId&gt;com.robin&lt;/groupId&gt;<br />    &lt;artifactId&gt;robin-cloud-oauth-server-9999&lt;/artifactId&gt;<br /><br /><br />    &lt;dependencies&gt;<br />        &lt;!--导⼊Eureka Client依赖--&gt;<br />        &lt;dependency&gt;<br />            &lt;groupId&gt;org.springframework.cloud&lt;/groupId&gt;<br />            &lt;artifactId&gt;spring-cloud-starter-netflix-eureka-client&lt;/artifactId&gt;<br />        &lt;/dependency&gt;<br />        &lt;!--导⼊spring cloud oauth2依赖--&gt;<br />        &lt;dependency&gt;<br />            &lt;groupId&gt;org.springframework.cloud&lt;/groupId&gt;<br />            &lt;artifactId&gt;spring-cloud-starter-oauth2&lt;/artifactId&gt;<br />            &lt;exclusions&gt;<br />                &lt;exclusion&gt;<br />                    &lt;groupId&gt;org.springframework.security.oauth.boot&lt;/groupId&gt;<br />                    &lt;artifactId&gt;spring-security-oauth2-<br />                        autoconfigure&lt;/artifactId&gt;<br />                &lt;/exclusion&gt;<br />            &lt;/exclusions&gt;<br />        &lt;/dependency&gt;<br />        &lt;dependency&gt;<br />            &lt;groupId&gt;org.springframework.security.oauth.boot&lt;/groupId&gt;<br />            &lt;artifactId&gt;spring-security-oauth2-autoconfigure&lt;/artifactId&gt;<br />            &lt;version&gt;2.1.11.RELEASE&lt;/version&gt;<br />        &lt;/dependency&gt;<br />        &lt;dependency&gt;<br />            &lt;groupId&gt;org.springframework.security.oauth&lt;/groupId&gt;<br />            &lt;artifactId&gt;spring-security-oauth2&lt;/artifactId&gt;<br />            &lt;version&gt;2.3.4.RELEASE&lt;/version&gt;<br />        &lt;/dependency&gt;<br />    &lt;/dependencies&gt;<br /><br />&lt;/project&gt;</pre>

**application.yml**

<pre class="wp-block-preformatted">server:<br />  port: 9999<br />spring:<br />  application:<br />    name: robin-cloud-oauth-server<br />eureka:<br />  client:<br />    service-url:<br />      defaultZone: http://robincloudeurekaservera:8761/eureka/,http://robincloudeurekaserverb:8762/eureka/ <em>#把eureka集群中的所有url都填写了进来，也可以只写⼀台，因为各个eureka server可以同步注册表<br />  </em>instance:<br />    <em>#使⽤ip注册，否则会使⽤主机名注册<br />    </em>prefer-ip-address: true<br />    instance-id: ${spring.cloud.client.ipaddress}:${spring.application.name}:${server.port}:@project.version@</pre>

**认证服务器配置类:**

<pre class="wp-block-preformatted">package com.robin.config;<br /><br />import org.springframework.beans.factory.annotation.Autowired;<br />import org.springframework.context.annotation.Configuration;<br />import org.springframework.http.HttpMethod;<br />import org.springframework.security.authentication.AuthenticationManager;<br />import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;<br />import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;<br />import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;<br />import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;<br />import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer;<br />import org.springframework.security.oauth2.provider.token.AuthorizationServerTokenServices;<br />import org.springframework.security.oauth2.provider.token.DefaultTokenServices;<br />import org.springframework.security.oauth2.provider.token.TokenStore;<br />import org.springframework.security.oauth2.provider.token.store.InMemoryTokenStore;<br /><br /><em>/**<br /> * </em><strong><em>@author </em></strong><em>Robin<br /> * </em><strong><em>@date </em></strong><em>2021/4/5<br /> *<br /> * 当前类为Oauth2 server的配置类（需要继承特定的⽗类<br /> * AuthorizationServerConfigurerAdapter）<br /> */<br /></em>@Configuration<br />@EnableAuthorizationServer // 开启认证服务器功能<br />public class OauthServerConfiger extends AuthorizationServerConfigurerAdapter {<br /><br />    @Autowired<br />    AuthenticationManager authenticationManager;<br /><br />    <em>/**<br />     * 认证服务器最终是以api接⼝的⽅式对外提供服务（校验合法性并⽣成令牌、校验令牌等）<br />     * 那么，以api接⼝⽅式对外的话，就涉及到接⼝的访问权限，需要在这⾥进⾏必要的配置<br />     *<br />     * ⽤来配置token端点的安全约束<br />     *<br />     * </em><strong><em>@param </em></strong><em>security<br />     * </em><strong><em>@throws </em></strong><em>Exception<br />     */<br />    </em>@Override<br />    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {<br />        super.configure(security);<br />        security.allowFormAuthenticationForClients() // 允许客户端表单验证<br />                .tokenKeyAccess("permitAll()") // 开启端⼝/oauth/token_key的访问权限<br />                .checkTokenAccess("permitAll()"); // 开启端⼝/oauth/check_token的访问权限<br />    }<br /><br />    <em>/**<br />     * 客户端详情配置，如client_id，secret<br />     * ⽤来配置客户端详情服务（ClientDetailsService），客户端详情信息在这⾥进⾏初始化<br />     * 客户端详情信息写死在这里或者是通过数据库来存储调取详情信息<br />     *<br />     * </em><strong><em>@param </em></strong><em>clients<br />     * </em><strong><em>@throws </em></strong><em>Exception<br />     */<br />    </em>@Override<br />    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {<br />        super.configure(clients);<br />        clients.inMemory() // 信息储存的地方，内存/数据库<br />                .withClient("client_robin") // // 添加⼀个client配置,指定其client_id<br />                .secret("123") // 指定客户端的密码/安全码<br />                .resourceIds("autodeliver") // 指定客户端所能访问资源id清单，此处的资源id与具体的资源服务器上的配置⼀样<br />                .authorizedGrantTypes("password","refresh_token") // 认证类型/令牌颁发模式，可以配置多个，但是不⼀定都⽤，具体使⽤哪种⽅式颁发token，需要客户端调⽤的时候传递参数指定<br />                .scopes("all"); // 客户端的权限范围<br />    }<br /><br />    <em>/**<br />     * 这⾥配置token令牌管理相关<br />     * token此时就是⼀个字串，当下的token需要在服务器端存储<br />     *<br />     * </em><strong><em>@param </em></strong><em>endpoints<br />     * </em><strong><em>@throws </em></strong><em>Exception<br />     */<br />    </em>@Override<br />    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {<br />        super.configure(endpoints);<br />        endpoints.tokenStore(tokenStore()) // 指定token的存储方法<br />                    .tokenServices(authorizationServerTokenServices())<br />                    .authenticationManager(authenticationManager) // 指定认证管理器<br />                    .allowedTokenEndpointRequestMethods(HttpMethod.<em>GET</em>, HttpMethod.<em>POST</em>);<br />    }<br /><br />    <em>/**<br />     *<br />     * 该⽅法⽤于创建tokenStore对象<br />     * token以什么形式存储， 这里是存在内存<br />     *<br />     * </em><strong><em>@return<br />     </em></strong><em>*/<br />    </em>public TokenStore tokenStore() {<br />        return new InMemoryTokenStore();<br />    }<br /><br />    public AuthorizationServerTokenServices authorizationServerTokenServices() {<br />        // 使⽤默认实现<br />        DefaultTokenServices defaultTokenServices = new DefaultTokenServices();<br />        defaultTokenServices.setSupportRefreshToken(true); // 是否开启token刷新<br />        defaultTokenServices.setTokenStore(tokenStore());<br />        // 设置token有效时间（⼀般设置为2个⼩时）<br />        // 这里设置了20秒<br />        defaultTokenServices.setAccessTokenValiditySeconds(20); // access_token就是我们请求资源需要携带的令牌<br />        // 设置刷新token的有效时间<br />        defaultTokenServices.setRefreshTokenValiditySeconds(259200); // 3天<br />        return defaultTokenServices;<br />    }<br />}<br /></pre>

关于TokenStore:

  * InMemoryTokenStore
      * 默认采⽤，它可以完美的⼯作在单服务器上（即访问并发量 压⼒不⼤的情况下，并且它在失败的时候不会进⾏备份），⼤多数的项⽬都可以使⽤这个版本的实现来进⾏尝试，你可以在开发的时候使⽤它来进⾏管理，因为不会被保存到磁盘中，所以更易于调试。
  * JdbcTokenStore
      * 这是⼀个基于JDBC的实现版本，令牌会被保存进关系型数据库。使⽤这个版本的实现时， 你可以在不同的服务器之间共享令牌信息，使⽤这个版本的时候请注意把&#8221;spring-jdbc&#8221;这个依赖加⼊到你的 classpath当中。
  * JwtTokenStore
      * 这个版本的全称是 JSON Web Token（JWT），它可以把令牌相关的数据进⾏编码（因此对于后端服务来说，它不需要进⾏存储，这将是⼀个重⼤优势），缺点就是这个令牌占⽤的空间会⽐较⼤，如果你加⼊了⽐较多⽤户凭证信息。

#### <span class="ez-toc-section" id="%E8%AE%A4%E8%AF%81%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%89%E5%85%A8%E9%85%8D%E7%BD%AE%E7%B1%BB"></span>**认证服务器安全配置类**<span class="ez-toc-section-end"></span> {.wp-block-heading}

<pre class="wp-block-preformatted">package com.robin.config;<br /><br />import org.springframework.beans.factory.annotation.Autowired;<br />import org.springframework.context.annotation.Bean;<br />import org.springframework.security.authentication.AuthenticationManager;<br />import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;<br />import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;<br />import org.springframework.security.core.userdetails.User;<br />import org.springframework.security.core.userdetails.UserDetails;<br />import org.springframework.security.crypto.password.NoOpPasswordEncoder;<br />import org.springframework.security.crypto.password.PasswordEncoder;<br /><br />import java.util.ArrayList;<br /><br /><em>/**<br /> * 该配置类，主要处理⽤户名和密码的校验等事宜<br /> *<br /> * </em><strong><em>@author </em></strong><em>Robin<br /> * </em><strong><em>@date </em></strong><em>2021/4/5<br /> */<br /></em>public class SecurityConfiger extends WebSecurityConfigurerAdapter {<br /><br />    <em>/**<br />     * 注册认证管理器对象到容器<br />     */<br />    </em>@Bean<br />    @Override<br />    public AuthenticationManager authenticationManagerBean() throws Exception {<br />        return super.authenticationManagerBean();<br />    }<br /><br />    <em>/**<br />     * 密码编码对象（密码不进⾏加密处理）<br />     * </em><strong><em>@return<br />     </em></strong><em>*/<br />    </em>@Bean<br />    public PasswordEncoder passwordEncoder() {<br />        return NoOpPasswordEncoder.<em>getInstance</em>();<br />    }<br /><br />    @Autowired<br />    private PasswordEncoder passwordEncoder;<br /><br />    <em>/**<br />     * 处理⽤户名和密码验证事宜<br />     * 1）客户端传递username和password参数到认证服务器<br />     * 2）⼀般来说，username和password会存储在数据库中的⽤户表中<br />     * 3）根据⽤户表中数据，验证当前传递过来的⽤户信息的合法性<br />     */<br />    </em>@Override<br />    protected void configure(AuthenticationManagerBuilder auth) throws Exception {<br />        // 在这个⽅法中就可以去关联数据库了，当前我们先把⽤户信息配置在内存中<br />        // 实例化⼀个⽤户对象(相当于数据表中的⼀条⽤户记录)<br />        UserDetails user = new User("admin","123456",new ArrayList&lt;&gt;());<br />        auth.inMemoryAuthentication()<br />                .withUser(user)<br />                .passwordEncoder(passwordEncoder);<br />    }<br />}<br /></pre>

### <span class="ez-toc-section" id="%E8%B5%84%E6%BA%90%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%88Resource_Server%EF%BC%89"></span>**资源服务器（Resource Server）**<span class="ez-toc-section-end"></span> {.wp-block-heading}

#### <span class="ez-toc-section" id="%E8%B5%84%E6%BA%90%E6%9C%8D%E5%8A%A1%E9%85%8D%E7%BD%AE%E7%B1%BB"></span>**资源服务配置类**<span class="ez-toc-section-end"></span> {.wp-block-heading}

<pre class="wp-block-preformatted">package com.robin.config;<br /><br />import org.springframework.security.config.annotation.web.builders.HttpSecurity;<br />import org.springframework.security.config.http.SessionCreationPolicy;<br />import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;<br />import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer;<br />import org.springframework.security.oauth2.provider.token.RemoteTokenServices;<br /><br /><em>/**<br /> * </em><strong><em>@author </em></strong><em>Robin<br /> * </em><strong><em>@date </em></strong><em>2021/4/5<br /> */<br /></em>public class ResourceServerConfiger extends ResourceServerConfigurerAdapter {<br /><br />    private String sign_key = "robin123"; // jwt签名密钥<br /><br />    <em>/**<br />     * 该⽅法⽤于定义资源服务器向远程认证服务器发起请求，进⾏token校验等事宜<br />     *<br />     * </em><strong><em>@param </em></strong><em>resources<br />     * </em><strong><em>@throws </em></strong><em>Exception<br />     */<br />    </em>@Override<br />    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {<br />        // 设置当前资源服务的资源id<br />        resources.resourceId("autodeliver");<br />        // 定义token服务对象（token校验就应该靠token服务对象）<br />        RemoteTokenServices remoteTokenServices = new RemoteTokenServices();<br />        // 校验端点/接⼝设置<br />        remoteTokenServices.setCheckTokenEndpointUrl("http://localhost:9999/oauth/check_token");<br />        // 携带客户端id和客户端安全码<br />        remoteTokenServices.setClientId("client_robin");<br />        remoteTokenServices.setClientSecret("123");<br />        resources.tokenServices(remoteTokenServices);<br />    }<br /><br />    <em>/**<br />     * 通过此方法设置API接口访问权限<br />     *<br />     * </em><strong><em>@param </em></strong><em>http<br />     * </em><strong><em>@throws </em></strong><em>Exception<br />     */<br />    </em>@Override<br />    public void configure(HttpSecurity http) throws Exception {<br />        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.<em>IF_REQUIRED</em>) // 设置session的创建策略（此处是根据需要创建）<br />                                .and()<br />                                .authorizeRequests()<br />                                // 正则表达式，/autodeliver和/demo为前缀的链接需要被验证<br />                                // 其他请求不验证<br />                                .antMatchers("/autodeliver/**").authenticated()<br />                                .antMatchers("/demo/**").authenticated()<br />                                .anyRequest().permitAll();<br />    }<br />}<br /></pre>

后续补充使⽤ JWT 进⾏改造，使⽤JWT机制之后资源服务器不需要访问认证服务。使用JWT将⽤户信息存储到token中，让客户端⼀直持有这个token，使token的验证也在资源服务器进⾏，这样避免资源服务器和认证服务器频繁的交互。

后续也会将相关代码上传至Github。