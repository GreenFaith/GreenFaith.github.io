---
layout: post
title: "python 异步网络编程扯淡"
date: 2014-06-11 14:23:00

---

@cmgs [Python网络编程乱弹琴](http://www.douban.com/note/341479741/)

```

洪荒年代，从quixote到django，网络这边的模型一直停留在同步模式，单进程单线程的处理各种请求。  

虽然我们一边嘲笑那个时候的ruby没有真系统线程性能还烂得狗一样，同时也被erlang这种，  
呃现在也是，碾压成翔。  

感谢GIL，即便有着stackless python这种erlang一级的神器，由于库的原因很少人会用，   
大家只能默默的撸着CPython心中一万头草泥马奔腾。   

所以在 Python 网络应用开发结构上，基本玩不出什么花样。

```
</br>   

先来谈谈CPython   

关于CPython似乎有太多的槽点，简单总结下 @蒋云鹏在知乎上的回答并扯淡      
<center>[在大型项目上，Python 是个烂语言吗？](http://www.zhihu.com/question/21017354/answer/18045229)</center>

```
1.python没有多线程，无法利用多核，也更别提强大的多线程包了。
```
</br>
在 GIL 大锁的淫威下，python 代码在运行时同时只能有一个进程在执行，所以不存在真正的并行。
执行 cpu bound 任务多线程的总性能比单线程还要低.   

然后我们有 `Twisted` 了。  

2000 的时候一位叫做 Glyph 的程序员在用 Java 做一个叫做 Twisted Reality 的 MUD 游戏，之前的 threads 实现太过于痛苦，然后他发现 Python 的 select 异步 IO 给力，于是就转用 Python 开发。然后世界上诞生了一个叫做 Twisted 的网络编程框架。 


<center>[The Architecture of Open Source Applications (Volume 2): Twisted](http://www.aosabook.org/en/twisted.html)</center>

正如文档中所说的，Twisted 做了许多在 python 当时网络平台中未曾有过的事。  
   
横跨平台，事件驱动，可替换驱动循环，实现了reactor模型，各种协议支持等等。   
而当时 java 平台还没有任何异步api支持(java.nio 在2002才被添加到java中).   

@cmgs 如是说   

```
爷爷辈们玩event-driven的时候，node.js的亲爹V8估摸着还没生出来……

到了Python在国内壮大的那几年，好吧也就是10年左右，网络层这块有有了新的选择，  
那就是Gevent/Eventlet亦或是tornado这样的callback hell   

```
所以其实写了这么多，GIL这个坑在web编程上并不是问题。正所谓塞翁失马,焉知非福。   

</br>  



```
2. python缺乏一系列诊断工具。jvm提供的jstack,jstat,btrace真是太强大了，而python...   
提一个基本需求吧：谁知道如何实时dump线上运行的python thread stack？
```

线上的服务挂了都没法直接attach诊断原因，而现在似乎已经有了[pyringe](https://github.com/google/pyringe)之类的诊断工具了。  
gdb+python debug 拓展 [在 Python 中使用 GDB 来调试](http://blog.log4d.com/2013/11/python-gdb/)   
更多工具查看 [我常用的Python调试工具](http://segmentfault.com/a/1190000000356018)   

</br>

```
3. python没有一个强大，高性能的web应用服务器。gunicorn和uwsgi已经算python最好的web   
应用服务器了，但性能真的差强人意。
```

Gevent loop engine  based  
性能这东西变量较多还真不好说:)  

</br>

```
4. 有人会反驳说python没有多线程，但有纤程（协程）啊！   
是的，但python如果要使用gevent，有太多的坑要踩。   
你除了自己的代码需要考虑纤程，还要考虑mysql,mongoDB,redis,memcached等客户端。

```
mongo已有motor这样的异步客户端可用.  

</br>

```
5. python大部分开源库的connection没有pool，或者是这些conection pool   
不能很好的和gevent一起工作，这在大型互联网开发中是很恐怖的。
原因是，很多conection pool是通过threadlocal实现的(参考python    
memcached的源代码（https://pypi.python.org/pypi/python-memcached/）)，  
而纤程的threadlocal实现有点奇葩。
```

