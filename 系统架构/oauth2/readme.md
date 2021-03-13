

# OAuth 2.0 Provider
> The OAuth 2.0 provider mechanism is responsible for exposing OAuth 2.0 protected resources. The configuration involves establishing the OAuth 2.0 clients that can access its protected resources independently or on behalf of a user. The provider does this by managing and verifying the OAuth 2.0 tokens used to access the protected resources. Where applicable, the provider must also supply an interface for the user to confirm that a client can be granted access to the protected resources (i.e. a confirmation page).

+ oauth2提供者机制是负责暴露oauth2受保护的资源
+ 配置建立oauth2客户端能够独立或代表用户访问受保护的资源
+ 管理和认证oauth2.0  tokens访问受保护的资源
+ 在适当情况下，提供者必须提供一个接口给用户，以确认客户端能访问受保护的资源


# OAuth 2.0 Provider Implementation
~~~
The provider role in OAuth 2.0 is actually split between Authorization Service and Resource Service, and while these sometimes reside in the same application, with Spring Security OAuth you have the option to split them across two applications, and also to have multiple Resource Services that share an Authorization Service. The requests for the tokens are handled by Spring MVC controller endpoints, and access to protected resources is handled by standard Spring Security request filters. The following endpoints are required in the Spring Security filter chain in order to implement OAuth 2.0 Authorization Server:

AuthorizationEndpoint is used to service requests for authorization. Default URL: /oauth/authorize.
TokenEndpoint is used to service requests for access tokens. Default URL: /oauth/token.
The following filter is required to implement an OAuth 2.0 Resource Server:

The OAuth2AuthenticationProcessingFilter is used to load the Authentication for the request given an authenticated access token.
For all the OAuth 2.0 provider features, configuration is simplified using special Spring OAuth @Configuration adapters. There is also an XML namespace for OAuth configuration, and the schema resides at https://www.springframework.org/schema/security/spring-security-oauth2.xsd. The namespace is http://www.springframework.org/schema/security/oauth2.
~~~




# Authorization Server Configuration