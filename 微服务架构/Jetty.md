通常我们进行Java Web项目开发，必须要选择一种服务器来部署并运行Java应用程序，Tomcat和Jetty作为目前全球范围内最著名的两款开源servlet容器，该怎么选呢。

Tomcat

Tomcat属于Apache项目下核心项目，是一个免费的开放源代码的Web 应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是很多Web项目开发的首选。

![img](http://pics7.baidu.com/feed/4bed2e738bd4b31c3b6b06f43ba0d4799f2ff8ac.jpeg?token=7983e4b63c74a1b4bbcada5755562a87)

Jetty

基于Java语言编写的一个开源servlet容器，为Jsp和servlet提供了运行环境，可以迅速为一些独立运行的Java应用提供网络和web连接。

![img](http://pics2.baidu.com/feed/4ec2d5628535e5dde3ff43f1c8b054e9cf1b624e.jpeg?token=379471033a64da92d636109a00c1d3ed)

异同点

相同点就是它们都是一种servlet引擎，都支持标准的servlet规范和JavaEE的规范。

**1、比较下Tomcat和Jetty的架构**

![img](http://pics0.baidu.com/feed/b3119313b07eca806898a7022d5564dba0448337.jpeg?token=afe5a6cd1a783eea3d042fb2308fd731)

Tomcat架构图

![img](http://pics7.baidu.com/feed/0bd162d9f2d3572cd96e19173665902160d0c3ef.jpeg?token=1e641e5891d4cd25d57f617c9ba1e65a)

Jetty架构图

Tomcat最顶层是Service，控制了服务器的整个生命周期，每一个Service由一个Container和多个Connector组成，形成一个独立完整的处理单元，对外请求。

Jetty的核心是Server，整体包含了多个Handle，还有一个Connector组成，Connector负责接受请求，将请求分配给一个队列去进行处理。

Jetty的架构设计要比tomcat的更清晰，简单。

**2、性能比较**

Jetty可以同时处理大量连接而且可以长时间保持连接，适合于web聊天应用等等。而且按需加载组件，减少了服务器的内存开销，内部默认采用NIO异步处理等特性提高了服务器性能。配置更加的灵活、简单、开发效率快、扩展性强。

从技术上来说的话，Jetty不是一个功能全面的J2EE服务器，缺少很多对J2EE功能的支持。

而Tomcat适合处理少数非常繁忙的链接，也就是说链接生命周期短的请求服务。它自身扩展了大量的J2EE特性来满足各种企业项目应用的需求，对于很多的Java web项目来说，我们可能并不需要这么多高级特性，从一定程度上浪费了很多的资源。

小结

说了那么多，总是在围绕Jetty比Tomcat强是吧。

但是总不能因为Tomcat性能比Jetty稍微差就否定它，不去使用它了。所以单纯的比较两者的性能没有很大的必要，关键是看使用场景，不同的场景是有差异的。

Jetty目前正在快速成长为一个优秀的 Servlet 引擎，但是Tomcat的地位还是没办法撼动，市场份额远远不及Tomcat的多。Tomcat自出道以来，经过这么长时间的发展，已经得到广泛企业的认可，所以比起Jetty更加的稳定和成熟，尤其是在企业应用方面成为了不二选择。

目前Jetty因为自身的技术优点，使得市场使用率和社区活跃性也在不断的提高。希望也做的越来越好吧。