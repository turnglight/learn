

# Spring Security Oauth2

## 核心概念[（理解Oauth2规范）](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
+ 资源服务器
+ 认证服务器
+ Security安全配置


## 四种认证场景

+ 授权码模式（authorization code）
+ 简化模式（implicit）
+ 密码模式（resource owner password credentials）
+ 客户端模式（client credentials）
> 常用的接口对接使用的是密码模式和客户端模式。授权码模式是最严密的调用模式，使用了回调地址，如第三方认证，qq第三方登录都是使用的这种模式。


## 开发配置步骤
+ 配置资源服务器
+ 配置认证服务器
+ 配置Spring Security


## 配置资源服务器

## 配置认证服务器
~~~java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {}
~~~

### **@EnableAuthorizationServer**
~~~
The @EnableAuthorizationServer annotation is used to configure the OAuth 2.0 Authorization
Server mechanism, together with any @Beans that implement AuthorizationServerConfigurer
(there is a handy adapter implementation with empty methods).
~~~
`@EnableAuthorizationServer`注解与一个实现了`AuthorizationServerConfigurer `的类一起使用，这个类需要实现三个特性方法，如下：

+ `ClientDetailsServiceConfigurer`: a configurer that defines the client details service. Client details can be initialized, or you can just refer to an existing store.
+ `AuthorizationServerSecurityConfigurer`: defines the security constraints on the token endpoint.
+ `AuthorizationServerEndpointsConfigurer`: defines the authorization and token endpoints and the token services.
~~~java
public class AuthorizationServerConfigurerAdapter implements AuthorizationServerConfigurer {
    public AuthorizationServerConfigurerAdapter() {}
    //配置AuthorizationServer安全认证的相关信息，创建ClientCredentialsTokenEndpointFilter核心过滤器
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {}
    //配置OAuth2的客户端相关信息
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {}
    //配置AuthorizationServerEndpointsConfigurer众多相关类，包括配置身份认证器，配置认证方式，TokenStore，TokenGranter，OAuth2RequestFactory
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {}
}
~~~

### 配置Spring Security
