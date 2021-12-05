# 前言

`Nginx` 是一个 **免费的**，**开源的**，**高性能** 的 `HTTP` 服务器和 **反向代理**，以及 `IMAP` / `POP3` 代理服务器。 `Nginx` 以其高性能，稳定性，丰富的功能，简单的配置和低资源消耗而闻名。`Nginx`是一个 `Web` 服务器，也可以用作 **反向代理**，**负载均衡器** 和 `HTTP` **缓存**。

很多高知名度的网站都使用 `Nginx`，如：`Netflix`，`GitHub`，`SoundCloud`，`MaxCDN` 等。

# NginX v.s. Apache

> ### Serving Static and Dynamic Content
>
> Another reason users are replacing Apache HTTP Server with NGINX is to adopt a new model for delivering applications made possible by NGINX. NGINX can deliver static content locally, but for dynamic content it acts as proxy in front of other servers that deliver the dynamic application content, thus keeping NGINX lean and leaving the generation of dynamic content to servers that specialize in it, such as FastCGI‑ or uwsgi‑based servers, application servers such as WebSphere, JBoss, and Tomcat, or even other web servers such as Apache.

# 正文

Nginx 解决了服务器的C10K（就是在一秒之内连接客户端的数目为10k即1万）问题。它的设计不像传统的服务器那样使用线程处理请求，而是一个更加高级的机制—**事件驱动**机制，是一种异步事件驱动结构。

**二、Nginx的特点**

- 跨平台：可以在大多数Unix like 系统编译运行。而且也有Windows的移植版本。 
- 配置异常简单：非常的简单，易上手。 
- 非阻塞、高并发连接：数据复制时，磁盘I/O的第一阶段是非阻塞的。官方测试能支持5万并发连接，实际生产中能跑2~3万并发连接数（得益于Nginx采用了最新的**epoll**事件处理模型（消息队列）。 
- Nginx代理和后端Web服务器间无需长连接； 
- Nginx接收用户请求是异步的，即先将用户请求全部接收下来，再一次性发送到后端Web服务器，极大减轻后端Web服务器的压力。 
- 发送响应报文时，是边接收来自后端Web服务器的数据，边发送给客户端。 
- 网络依赖性低，理论上只要能够ping通就可以实施负载均衡，而且可以有效区分内网、外网流量。 
- 支持内置服务器检测。Nginx能够根据应用服务器处理页面返回的状态码、超时信息等检测服务器是否出现故障，并及时返回错误的请求重新提交到其它节点上。 
- 此外还有内存消耗小、成本低廉（比F5硬件负载均衡器廉价太多）、节省带宽、稳定性高等特点。

**三、Nginx的整体架构**

![img](https://segmentfault.com/img/remote/1460000022476839)

**1、模块化设计**

Nginx 的Worker 进程，包括核心和功能性模块 ，核心模块负责维持一个运行循环（ run-loop ），执行网络请求处理的不同阶段的模块功能，比如：存储读写、内容传输 、网络读写 、外出过滤 ，以及将请求发往上游服务器等。而其代码的模块化设计 ，也使得我们可以根据需要对功能模块进行适当的选择和修改 ，编译成具有特定功能的服务器。

**2、代理设计**

代理（proxy）设计，可以说是 Nginx 深入骨髓的设计，无论是对于HTTP，还是对于Memcache 、Redis、FastCGI 等的网络请求或响应，本质上都采用了代理机制 。所以，Nginx 天生就是高性能的代理服务器 。

**3、事件驱动模型**

基于异步及非阻塞的事件驱动模型 ，可以说是 Nginx 得以获得 高并发 、 高性能 的关键因素，同时也得益于对 Linux 、 Solaris 及类 BSD 等操作系统内核中 事件通知 及 I/O 性能增强功能 的采用，如kqueue 、 epoll 及 event ports 。

**4、主进程模型**

Nginx 启动时，会生成两种类型的进程，一个是主进程 （ Master ）， 一个或多个工作进程 （ Worker ）。主进程并不处理网络请求，主要负责调度工作进程 ，也就是图示的3项：加载配置 、 启动工作进程及非停升级。所以Nginx启动以后，查看操作系统的进程列表，我们就能看到至少有两个Nginx进程。

**5、工作进程模型**

服务器实际处理网络请求及响应的是工作进程，在类Unix 系统上，Nginx可以配置多个Worker ，而每个Worker 进程都可以同时处理数以千计的网络请求。

**四、Nginx的模块化设计**

高度模块化的设计是 Nginx 的架构基础。Nginx 服务器被分解为多个模块 ，每个模块就是一个功能模块 ，只负责自身的功能，模块之间严格遵循 “高内聚，低耦合” 的原则。

![img](https://segmentfault.com/img/remote/1460000022476841)

**1、核心模块**

核心模块是 Nginx 服务器正常运行必不可少的模块，提供错误日志记录 、配置文件解析、事件驱动机制、进程管理等核心功能。

**2、标准HTTP模块**

标准 HTTP 模块提供 HTTP 协议解析相关的功能，比如： 端口配置 、 网页编码设置 、 HTTP响应头设置 等等。

**3、可选HTTP模块**

可选 HTTP 模块主要用于 扩展 标准的 HTTP 功能，让 Nginx 能处理一些特殊的服务，比如：Flash 多媒体传输 、解析 GeoIP 请求、 网络传输压缩 、 安全协议 SSL 支持等。

**4、邮件服务模块**

邮件服务模块主要用于支持 Nginx 的 邮件服务 ，包括对 POP3 协议、 IMAP 协议和 SMTP协议的支持。

**5、第三方模块**

第三方模块是为了扩展 Nginx 服务器应用，完成开发者自定义功能，比如：Json 支持、 Lua 支持等。

**五、代理设计中的正向代理和反向代理**

首先，代理服务器一般指局域网内部的机器通过代理服务器发送请求到互联网上的服务器，代理服务器一般作用在客户端。例如：GoAgent翻墙软件。我们的客户端在进行翻墙操作的时候，我们使用的正是正向代理，通过正向代理的方式，在我们的客户端运行一个软件，将我们的HTTP请求转发到其他不同的服务器端，实现请求的分发。

![img](https://segmentfault.com/img/remote/1460000022476842)

反向代理服务器作用在服务器端，它在服务器端接收客户端的请求，然后将请求分发给具体的服务器进行处理，然后再将服务器的相应结果反馈给客户端。Nginx就是一个反向代理服务器软件。

![img](https://segmentfault.com/img/remote/1460000022476844)

从上图可以看出：客户端必须设置正向代理服务器，当然前提是要知道正向代理服务器的IP地址，还有代理程序的端口。 
反向代理正好与正向代理相反，对于客户端而言代理服务器就像是原始服务器，并且客户端不需要进行任何特别的设置。客户端向反向代理的命名空间（name-space）中的内容发送普通请求，接着反向代理将判断向何处（原始服务器）转交请求，并将获得的内容返回给客户端。

![img](https://segmentfault.com/img/remote/1460000022476843)

**六、Nginx事件驱动模型**

在 Nginx 的异步非阻塞机制中，工作进程在调用 IO 后，就去处理其他的请求，当 IO 调用返回后，会通知该工作进程 。对于这样的系统调用，主要使用 Nginx 服务器的事件驱动模型来实现。

![img](https://segmentfault.com/img/remote/1460000022476846)

如上图所示， Nginx 的 事件驱动模型由事件发送器、事件收集器和事件处理器三部分基本单元组成：

- 事件发送器：负责将 IO 事件发送到事件处理器 ；
- 事件收集器：负责收集Worker 进程的各种 IO 请求；
- 事件处理器：负责各种事件的响应工作 。

事件发送器将每个请求放入一个 待处理事件列表 ，使用非阻塞 I/O 方式调用 事件处理器来处理该请求。其处理方式称为 “多路 IO 复用方法” ，常见的包括以下三种：select 模型、 poll模型、 epoll 模型。

**七、Nginx的请求方式处理**

Nginx 是一个高性能的 Web 服务器，能够同时处理大量的并发请求。它结合多进程机制和异步机制 ，异步机制使用的是异步非阻塞方式 ，接下来就给大家介绍一下 Nginx 的多线程机制和异步非阻塞机制 。

**1、多进程机制**

服务器每当收到一个客户端时，就有 服务器主进程 （ master process ）生成一个 子进程（ worker process ）出来和客户端建立连接进行交互，直到连接断开，该子进程就结束了。

使用进程的好处是各个进程之间相互独立，不需要加锁，减少了使用锁对性能造成影响，同时降低编程的复杂度，降低开发成本。其次，采用独立的进程，可以让进程互相之间不会影响 ，如果一个进程发生异常退出时，其它进程正常工作， master 进程则很快启动新的 worker 进程，确保服务不会中断，从而将风险降到最低。

缺点是操作系统生成一个子进程需要进行 内存复制等操作，在资源和时间上会产生一定的开销。当有大量请求时，会导致系统性能下降 。

**2、异步非阻塞机制**

每个工作进程 使用 异步非阻塞方式 ，可以处理 多个客户端请求 。

当某个 工作进程 接收到客户端的请求以后，调用 IO 进行处理，如果不能立即得到结果，就去 处理其他请求 （即为 非阻塞 ）；而 客户端 在此期间也 无需等待响应 ，可以去处理其他事情（即为 异步 ）。

当 IO 返回时，就会通知此 工作进程 ；该进程得到通知，暂时 挂起 当前处理的事务去 响应客户端请求 。

**八、Nginx进程处理模型**

Nginx 服务器使用 master/worker 多进程模式 。多线程启动和执行的流程如下：

- 主程序 Master process 启动后，通过一个 for 循环来 接收 和 处理外部信号 ；
- 主进程通过 fork() 函数产生 worker 子进程 ，每个 子进程 执行一个 for 循环来实现 Nginx 服务器 对事件的接收 和 处理 。

![img](https://segmentfault.com/img/remote/1460000022476845)

一般推荐 worker 进程数与CPU内核数一致，这样一来不存在大量的子进程生成和管理任务，避免了进程之间竞争CPU 资源和进程切换的开销。而且 Nginx 为了更好的利用 多核特性 ，提供了 CPU 亲缘性的绑定选项，我们可以将某一个进程绑定在某一个核上，这样就不会因为进程的切换带来 Cache 的失效。

对于每个请求，有且只有一个工作进程 对其处理。首先，每个 worker 进程都是从 master进程 fork 过来。在 master 进程里面，先建立好需要 listen 的 socket（listenfd） 之后，然后再 fork 出多个 worker 进程。

所有 worker 进程的 listenfd 会在新连接到来时变得可读 ，为保证只有一个进程处理该连接，所有 worker 进程在注册 listenfd 读事件前抢占 accept_mutex ，抢到互斥锁的那个进程注册 listenfd 读事件 ，在读事件里调用 accept 接受该连接。

当一个 worker 进程在 accept 这个连接之后，就开始读取请求、解析请求、处理请求，产生数据后，再返回给客户端 ，最后才断开连接。这样一个完整的请求就是这样的了。我们可以看到，一个请求，完全由 worker 进程来处理，而且只在一个 worker 进程中处理。

![img](https://segmentfault.com/img/remote/1460000022476847)

在 Nginx 服务器的运行过程中， 主进程和工作进程 需要进程交互。交互依赖于 Socket 实现的管道来实现。

**1、主进程与工作进程交互**

这条管道与普通的管道不同，它是由主进程指向工作进程的单向管道，包含主进程向工作进程发出的指令，工作进程 ID 等；同时主进程与外界通过信号通信；每个 子进程具备接收信号，并处理相应的事件的能力。

**2、工作进程与工作进程交互**

这种交互是和主进程-工作进程交互是基本一致的，但是会通过主进程间接完成。 工作进程之间是相互隔离的，所以当工作进程 W1 需要向工作进程 W2 发指令时，首先找到 W2 的 进程ID，然后将正确的指令写入指向 W2 的 通道。W2 收到信号采取相应的措施。

**九、小结**

通过这篇文章，我们对 Nginx 服务器的整体架构有了一个整体的认识。包括其模块化的设计、多进程和异步非阻塞的请求处理方式、事件驱动模型等。通过这些理论知识，才能更好地领悟 Nginx 的设计思想。对于我们学习 Nginx 来说有很大的帮助。



# Nginx 负载均衡 策略

## 1、轮询（默认）

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

```
upstream backserver {
    server 192.168.0.14;
    server 192.168.0.15;
}
```

## 2、weight

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的
情况。

```
upstream backserver {
    server 192.168.0.14 weight=3;
    server 192.168.0.15 weight=7;
}
```

权重越高，在被访问的概率越大，如上例，分别是30%，70%。

## 3、ip_hash

上述方式存在一个问题就是说，在负载均衡系统中，假如用户在某台服务器上登录了，那么该用户第二次请求的时候，因为我们是负载均衡系统，每次请求都会重新定位到服务器集群中的某一个，那么***已经登录某一个服务器的用户再重新定位到另一个服务器，其登录信息将会丢失，这样显然是不妥的\***。
我们可以采用***ip_hash\***指令解决这个问题，如果客户已经访问了某个服务器，当用户再次访问时，会将该请求通过***哈希算法，自动定位到该服务器\***。
每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决***session的问题\***。

```
upstream backserver {
    ip_hash;
    server 192.168.0.14:88;
    server 192.168.0.15:80;
}
```

## 4、fair（第三方）

按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```
upstream backserver {
    server server1;
    server server2;
    fair;
}
```

## 5、url_hash（第三方）

按访问url的hash结果来分配请求，使每个url定向到同一个（对应的）后端服务器，后端服务器为缓存时比较有效。

```
upstream backserver {
    server squid1:3128;
    server squid2:3128;
    hash $request_uri;
    hash_method crc32;
}
```

在需要使用负载均衡的server中增加

```
proxy_pass http://backserver/; 
upstream backserver{ 
    ip_hash; 
    server 127.0.0.1:9090 down; (down 表示单前的server暂时不参与负载) 
    server 127.0.0.1:8080 weight=2; (weight 默认为1.weight越大，负载的权重就越大) 
    server 127.0.0.1:6060; 
    server 127.0.0.1:7070 backup; (其它所有的非backup机器down或者忙的时候，请求backup机器) 
} 
```

max_fails ：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误

fail_timeout:max_fails次失败后，暂停的时间
配置实例：

```
#user  nobody;

worker_processes  4;
events {
# 最大并发数
worker_connections  1024;
}
http{
    # 待选服务器列表
    upstream myproject{
        # ip_hash指令，将同一用户引入同一服务器。
        ip_hash;
        server 125.219.42.4 fail_timeout=60s;
        server 172.31.2.183;
    }

    server{
        # 监听端口
        listen 80;
        # 根目录下
        location / {
        # 选择哪个服务器列表
            proxy_pass http://myproject;
        }

    }
```