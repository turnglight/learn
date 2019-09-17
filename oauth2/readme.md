

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
~~~java
public class ResourceServerConfigurerAdapter implements ResourceServerConfigurer {
    public ResourceServerConfigurerAdapter() {
    }

    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
    }

    public void configure(HttpSecurity http) throws Exception {
    }
}
~~~
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


### 自定义认证失败响应结果

~~~java
@Configuration
@EnableResourceServer
class ResourceServerConfiguration extends ResourceServerConfigurerAdapter {
  @Override
    public void configure(ResourceServerSecurityConfigurer resources) {
        resources
            .resourceId(AuthorizationServerConfiguration.RESOURCE_ID)
            .stateless(true)
            //自定义认证异常
            .authenticationEntryPoint(new AuthExceptionEntryPoint())
            //自定义拒绝访问
            .accessDeniedHandler(new CustomAccessDeniedHandler());
    }
}
~~~

~~~java
package cloud.zuul.server.oauth2;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.oauth2.common.exceptions.InvalidTokenException;
import org.springframework.security.web.AuthenticationEntryPoint;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

public class AuthExceptionEntryPoint implements AuthenticationEntryPoint
{

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException) throws ServletException {
        Map<String, Object> map = new HashMap<String, Object>();
        Throwable cause = authException.getCause();
        if(cause instanceof InvalidTokenException) {
            map.put("code", 401);//401
            map.put("msg", "无效的token");
        }else{
            map.put("code", 401);//401
            map.put("msg", "访问此资源需要完全的身份验证");
        }
        map.put("data", authException.getMessage());
        map.put("success", false);
        map.put("path", request.getServletPath());
        map.put("timestamp", String.valueOf(new Date().getTime()));
        response.setContentType("application/json");
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        try {
            ObjectMapper mapper = new ObjectMapper();
            mapper.writeValue(response.getOutputStream(), map);
        } catch (Exception e) {
            throw new ServletException();
        }
    }
}
~~~

~~~java
package cloud.zuul.server.oauth2;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.web.access.AccessDeniedHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

@Component("customAccessDeniedHandler")
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response,
                       AccessDeniedException accessDeniedException)
            throws IOException, ServletException {
        response.setContentType("application/json;charset=UTF-8");
        Map<String,Object> map = new HashMap<String,Object>();
        map.put("code", 401);//401
        map.put("msg", "权限不足");
        map.put("data", accessDeniedException.getMessage());
        map.put("success", false);
        map.put("path", request.getServletPath());
        map.put("timestamp", String.valueOf(new Date().getTime()));
        ObjectMapper mapper = new ObjectMapper();
        response.setContentType("application/json");
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.getWriter().write(mapper.writeValueAsString(map));
    }
}

