
```
初始进程:src/core/nginx.c:main
    初始化主循环:src/core/ngx_cycle.c:ngx_init_cycle
        
        创建线程模块:src/core/ngx_thread_pool.c:ngx_thread_pool_create_conf   (rv = module->create_conf(cycle))
        
        读配置文件:src/core/ngx_conf_file.c:ngx_conf_parse
            src/core/ngx_conf_file.c:ngx_conf_handler
                加载ModSecurity配置项:ngx_http_modsecurity_enable
                创建线程池:src/core/ngx_thread_pool.c:ngx_thread_pool_add
            ngx_http_block
            src/core/ngx_conf_file.c:ngx_conf_parse
            ngx_http_core_location
    
        初始化线程池:src/core/ngx_thread_pool.c:ngx_thread_pool_init_conf   (module->init_conf(cycle, cycle->conf_ctx[cycle->modules[i]->index])

        开启监听端口:src/core/ngx_connection.c:ngx_open_listening_sockets

    启动master进程:src/os/unix/ngx_daemon.c:ngx_daemon

```

```
master进程
    开始进行master进程处理:src/os/unix/ngx_process_cycle.c:ngx_master_process_cycle
        启动worker进程:src/os/unix/ngx_process_cycle.c:ngx_worker_process


    接收并处理signal

```


```
worker线程
    开始进行worker进程处理:src/os/unix/ngx_process_cycle.c:ngx_worker_process_cycle
        初始化worker进程:src/os/unix/ngx_process_cycle.c:ngx_worker_process_init
            设置cpu affinity:src/os/unix/ngx_setaffinity.c:ngx_setaffinity
            
            初始化事件模块:src/event/ngx_event.c:ngx_event_process_init   (cycle->modules[i]->init_process(cycle) == NGX_ERROR)
                添加accept监听事件:src/event/ngx_epoll_module.c:ngx_epoll_add_event
            
            初始线程池模块:src/core/ngx_thread_pool.c:ngx_thread_pool_init_worker   (cycle->modules[i]->init_process(cycle) == NGX_ERROR)
                创建线程:src/core/ngx_thread_pool.c:ngx_thread_pool_cycle
            初始化ModSecurity:ngx_http_modsecurity_init
    

        开始事件循环:src/event/ngx_event.c:ngx_process_events_and_timers
            
            获取accept 互斥量:src/event/ngx_event_accept.c:ngx_trylock_accept_mutex
                没有获取到则从事件列表中删除accept监听事件:src/event/ngx_event_accept.c:ngx_disable_accept_events
            
            监听事件:src/event/modules/ngx_epoll_module.c:ngx_process_events/ngx_epoll_process_events
                开始监听事件:epoll_wait
                接收到一个事件
                    accept事件:src/evet/ngx_event_accept.c:ngx_event_accept     (rev->handler(rev);)
                        从连接池中获取一个连接:src/core/ngx_connection.c:ngx_get_connection
                        初始化连接:src/http/ngx_http_request.c:ngx_http_init_connection     (ls->handler(c);)
                            为新建连添加读事件:src/event/ngx_event.c:ngx_handle_read_event
                                ngx_add_event
                
                    request事件:src/http/ngx_http_request.c     (rev->handler(rev);)
                        开始接收请求:src/os/unix/ngx_recv.c:ngx_unix_recv       n = c->recv(c, b->last, size);
                        创建一个请求:src/http/ngx_http_request.c:ngx_http_create_request
                        处理请求行:src/http/ngx_http_request.c:ngx_http_process_request_line
                            读请求头，如果已经读到则退出:src/http/ngx_http_request.c:ngx_http_read_request_header
                            解析http请求:src/http/ngx_http_parse.c:ngx_http_parse_request_line
                            处理uri:src/http/nginx_http_request.c:ngx_http_process_request_uri
                            处理请求头:src/http/nginx_http_request.c:ngx_http_process_request_headers
                                解析http请求头部:src/http/ngx_http_parse.c:ngx_http_parse_header_line
                                处理请求:src/http/ngx_http_request.c:ngx_http_process_request


                处理accept后序操作:src/event/ngx_event_posted.c:ngx_event_process_posted
                处理后序操作:src/event/ngx_event_posted.c:ngx_event_process_posted


```





#Q
- Q1. 向一个epoll注册多个相同的fd会怎样？