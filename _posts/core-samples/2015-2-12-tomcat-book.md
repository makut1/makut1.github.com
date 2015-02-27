---
title: tomcat
layout: post
tagline: "Supporting tagline"
tags : [tomcat]
---
{% include JB/setup %}


**tomcat**

.
tomcat记录：请求行：请求类型（通常是get或是post），被请求对象的路径与名称，以及浏览器预备使用的http协议的最高版本标号。如果url不包括文件名，则浏览器必须传递/,它会转换为对网站默认网页的请求。GET /t/170347 HTTP/1.1 Host: www.v2ex.com 。$开头的则表示输入的命令要启动程序.用requestDUmperValue来查看更详细的web通信流量，这是tomcat提供的一个工具，只要在server.xml中移除注释符号，或者在host或context中加入一段代码就可以.'

>tomcat启动一闪而过的原因：在这台机器上，在eclipse中配置了tomcat所使用的jdk为1.7版本的，而在本机上用的jdk为1.6的，而且在server.xml中会自动更新<context>内容，因为这是eclipse给造成的。在eclipse中把tomcat中所配置的项目都给移除掉，在启动tomcat时，就可以再启动，一闪而过的问题就给解决了。这可能就是jdk的版本不同造成的。

>hibernate的对象有三种状态：
> 1.瞬时状态（transient） 2）持久态（persistent） 3.托管态
><a>  主键生成机制由10种<session-factory> c3p0数据库连接池配置，配置上其中的属性来实现其功能。hibernate有哪五个核心接口？他们的作用分别是什么？（session，sessionfactory，configur，transac，query和criteria）这几个核心的接口，也是比较常见的接口。 hibernate的对象检索方式有哪些？有oid检索方式，hql检索方式，qbc检索方式，本地sql检索方式。</a>
  
- &emsp;hibernate其他的功能：保存，更新和删除实体对象。调用存储过程。
- &emsp;什么是延迟加载？为什么要进行延迟加载？延迟加载：只在一个对象调用他的一对对或多对多关系时才将关系对象读取出来，这个过程对开发者来说是透明的，而且只进行了很少的数据库操作请求，因此会得到比较明显的性能提升。延迟加载的三种情况：
实体对象的延迟加载；集合类型的延迟加载；延迟属性加载。
- &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[不断行的空白格&nbsp;或&#160] 什么是缓存机制？什么是缓存？缓存是位于应用程序与物理数据源之间，用于临时存放复制数据的内存区域，目的是为了减少应用程序对物理数据源访问的次数，从而提高应用程序的运行性能。hibernate在查询数据时，首先到缓存中去查找，如果找到就这直接使用，找不到的时候就会从物理数据源中检索，所以把频繁使用的数据加载到缓存区后，就可以大大减少应用程序对物理数据源的访问，使得程序的运行性能明显地提升。

 ![](http://i.imgur.com/99G8YpU.png)

2015/2/13 17:53:12 
多得多[打砖块游戏404](http://lubie.co/404/)
*你想到的方法或是功能，用代码来实现，也许就会知道怎么去编代码*