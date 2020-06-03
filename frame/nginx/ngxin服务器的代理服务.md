
## 正向代理与反向代理的概念
+ 正向代理
    + 局域网内部服务访问Internet外部服务器时，通过代理服务进行访问监控与管理的场景为正向代理（局域网可直接访问外网，但外网不能直接访问内网）
+ 反向代理
    + 相反，Internet外部服务器访问局域网内部资源时，称之为反向代理

### 正向代理服务
+ 实际应用中，使用nginx服务器正向代理服务功能的情况很少，主要涉及到3条指令
    + resolver(指定DNS服务器IP地址)
    + resolver_timeout
    + proxy_pass(设置代理服务器的协议和地址，它不仅用于正向代理，更主要的是用于反向代理)
        + 语法结构为: proxy_pass URL;在代理服务器中，该指令配置为： proxy_pass http://$http_host$request_uri
### 反向代理服务
+ 反向代理服务主要有以下21条指令
    + proxy_pass
    + proxy_hide_header
    + proxy_pass_header
    + proxy_pass_request_body
    + proxy_pass_request_headers
    + proxy_set_header
    + proxy_set_body
    + proxy_bind
    + proxy_connect_timeout
    + proxy_read_timeout
    + proxy_send_timeout
    + proxy_http_version
    + proxy_method
    + proxy_ignore_client_abort
    + proxy_ignore_headers
    + proxy_redirect
    + proxy_intercept_errors
    + proxy_headers_hash_max_size
    + proxy_headers_hash_bucket_size
    + proxy_next_upstream
    + proxy_ssl_session_reuse

`proxy buffer`与`proxy cache`虽然都是用于提供IO吞吐效率，但它们是两个不同的概念，一个是缓冲，一个是缓存。buffer实现了相应数据的异步传输，cache主要实现了nginx服务器对客户端请求的快速响应。特别需要说明是的`proxy cache依赖proxy buffer`,只有在buffer开启的情况下，cache才能发挥作用。Nginx服务器在接收到被代理服务器的响应数据时，一方面通过proxy buffer将数据返回给客户端；另一方面通过proxy cache将响应数据在磁盘中进行缓存。在下一次接收到客户端的相同请求时，可以直接从磁盘缓存数据中获取响应数据，接收客户端的请求时间。
+ proxy buffer(缓冲)
    + proxy_buffering
    + proxy_buffers
    + proxy_buffer_size
    + proxy_busy_buffers_size
    + proxy_temp_path
    + proxy_max_temp_file_size
    + proxy_temp_file_write_size
+ proxy cache(缓存)
    + proxy_cache
    + proxy_cache_bypass
    + proxy_cache_key
    + proxy_cache_lock
    + proxy_cache_lock_timeout
    + proxy_cache_min_uses
    + proxy_cache_path
    + proxy_cache_use_stale
    + proxy_cache_valid
    + proxy_no_cache
    + proxy_store
    + proxy_store_access
    + 

