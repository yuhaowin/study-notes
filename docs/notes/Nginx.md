# 01 | 这一次，让我们一起来搞懂 NGINX
余浩 2019-06-19
![](http://ww1.sinaimg.cn/large/006tNc79gy1g47es2kqipj31i40u07wh.jpg)

#### 正向代理和反向代理

![正向代理](http://ww1.sinaimg.cn/large/006tNc79ly1g4218436hej30tm0f6dh4.jpg)

![反向代理](http://ww1.sinaimg.cn/large/006tNc79ly1g4218um92kj30su0f0400.jpg)

所谓正向、反向其实就是代理的对象不同，正向代理的代理对象是客户端,反向代理的代理对象是服务端，代理服务器在客户端那边就是正向代理，代理服务器在原始服务器那边就是反向代理。

用户在一次访问诉求中是有可能同时经历正向代理和反向代理的， **假设我在公司的内网环境中访问不了百度，此时可以通过一个代理(正向代理)服务器来访问百度，当这个代理服务器将我的请求转发给百度时，这个代理服务器对百度而言就是一个客户端，同时百度可能会有一个前置的反向代理服务器，将我的请求转发给一个真正可以处理我的请求的应用服务器，最终将结果返回给我。那在这整个链路中就即经历了正向代理又经历了反向代理。**

[正向代理和反向代理参考资料](https://www.jianshu.com/p/ae76c223c6ef)

******

nginx的主要使用场景

+ 静态资源服务
+ 反向代理服务
	+ 负载均衡
	+ 缓存加速	 
+ api服务 

> 应用服务一般运行效率低并发性 qps 都很差，需要很多个应用服务做集群，这时就需要 nginx 有反向代理功能，讲用户的请求传导给应用服务。在很多应用服务组成的集群环境中，需要做到动态扩容和熔灾，此时就要求 nginx 具有负载均衡功能。同时为了减少请求的响应时间，可以讲一些在一段时间不变的动态内容通过 nginx 缓存起来，以加速访问。



#### nginx 的组成部分

![nginx的4个组成部分](http://ww3.sinaimg.cn/large/006tNc79gy1g40115hawrj30vw0fwdic.jpg)

+ Nginx 二进制文件相当汽车本身，提供各种功能。
+ Nginx.conf 相当于老司机，合理有序的控制 Nginx 的功能。
+ access.log 记录行驶轨迹，（记录 http 的请求和响应）
+ error.log  记录错误日志

#### nginx 编译安装

不建议直接使用yun install 直接安装，推荐编译安装，编译安装可以定制化自己需要的模块

```shell
  ./configure --help    //可以看到完整的编译参数
```
[nginx官网](https://nginx.org/) 下载nginx源码包

##### 解压源码包

![解压后目录](http://ww4.sinaimg.cn/large/006tNc79gy1g40j09k0ayj30ue0eoaaq.jpg)

+ conf 配置文件的示例文件目录

+ contrib 有一个vim工具, 负责高亮 nginx 的语法 

```shell
  cp -r contrib/vim/*  ~/.vim/  //拷贝到自己的 vim 目录下
```

+ src 是 nginx 源代码目录

##### 指定 nginx 的安装目录

```shell
./configure --prefix=/Users/yuhao/Desktop/nginx-test
```

![执行 configure 命令](http://ww2.sinaimg.cn/large/006tNc79gy1g40jpto3w5j30r206aq31.jpg)

执行完后会生成中间文件 objs

![生成的中间文件objs](http://ww4.sinaimg.cn/large/006tNc79gy1g40jladz4ej30ya0dojs7.jpg)


**ngx_modules.c 决定的在接下来的编译过程中有哪些模块会被编译到 nginx 中**

##### make 编译

![执行 make 命令](http://ww4.sinaimg.cn/large/006tNc79gy1g40jr7qoi7j30yk08i0t4.jpg)

make 编译后生成的二进制文件依然在 objs 目录下，编译时生成的中间文件都是在 objs/src 目录下。

![执行 make 后的目录结构](http://ww3.sinaimg.cn/large/006tNc79gy1g40nq9iaalj30wi0d0js4.jpg)

##### make install 安装

会将 nginx 安装到 --prefix 指定的目录中，我这里是
`/Users/yuhao/Desktop/nginx-test` 

![执行 make install 命令](http://ww4.sinaimg.cn/large/006tNc79gy1g40nxkflwdj30y80aitaf.jpg)

![nginx 的安装目录](http://ww1.sinaimg.cn/large/006tNc79gy1g40o0buwr1j316s0o8wh1.jpg)

+ conf 配置文件示例目录，从源码中直接拷贝过来的。
+ html 标准网页示例目录。
+ logs 存放日志目录。
+ sbin 存放 nginx 二进制文件目录。

在以上编译好的自己的 nginx 中已经包含了我们自己指定的模块，但是每个模块的配置都有自己的配置语法。

![nginx 配置语法](http://ww3.sinaimg.cn/large/006tNc79gy1g41lnykx9tj31dx0u0ndf.jpg)

配置主要由指令和指令块组成

![配置例子](http://ww2.sinaimg.cn/large/006tNc79gy1g41lrjj8j8j31l50u049i.jpg)

![nginx 命令行](http://ww1.sinaimg.cn/large/006tNc79gy1g41lyh162zj30wq0kaad2.jpg)


nginx -s，是向 nginx 进程发信号的工具

linux文件系统中，改名并不会影响已经打开文件的写入操作，内核inode不变。



热部署

![](http://ww4.sinaimg.cn/large/006tNc79gy1g41m332rd3j313e0j0tem.jpg)

会启动一个新的 master进程，这新的 master 进程使用的是 新的 nginx 二进制文件启动的，新的master 会生成新的worker 老的 worker 也在运行


#### nginx 请求处理流程

![请求处理流程](http://ww4.sinaimg.cn/large/006tNc79gy1g46hent1qkj30v20e8dhm.jpg)

了解 nginx 是怎么记录日志 处理静态资源 反向代理。

三种流量会交由 nginx 处理，三种状态机分别处理这三种请求

TCP/UDP 交由四层传输层状态机

应用层 交由 HTTP 状态机

邮件 交由 Mail 状态机

nginx 使用的是非阻塞的事件驱动的处理方式，异步处理的方式都是通过状态机进行识别和处理的。

在做反向代理时，可以通过协议/应用层的协议将请求透传到上游服务。


#### nginx 进程结构

nginx 有两种进程结构，单进程和多进程，默认使用的是多进程结构，以便于 nginx 利用多核的特性

![进程结构](http://ww1.sinaimg.cn/large/006tNc79gy1g46hh18tmjj30q00f4myk.jpg)

在 nginx 的进程模型中，有一个 master 进程，该 master 进程有许多 子进程，众多子进程中分为两种，一种是 worker 进程，一种是 cache 相关的进程。

多个 worker 进程，是为了希望一个进程对应一个cpu核心，减少进程间的切换，一般 worker 的个数和cpu核心数一致，同时将 worker 进程和cpu核心绑定。确保一个进程从始至终都和一个核心绑定。可以更好的使用每个cpu核心上的缓存，减少缓存失效。

nginx 为什么是多进程结构，而不是多线程结构，因为线程之间是共享同一个地址空间的，如果出现了地址空间的越界时，会导致整个服务挂掉。

nginx 的父子进程之间的通信是通过信号进行的。


ps -ef | grep nginx

nginx -s reload

会让老的 worker 进程退出，在根据配置文件启动新的子进程

kill -SIGHUP 父进程的pid  == nginx -s reload

nginx 是一个多进程的程序，多进程之间的通信可以使用共享内存、信号等，nginx 在做进程间的管理时只使用信号

能够发送、处理信号的有 master 进程、worker 进程、nginx 的命令

![nginx 的信号管理](http://ww2.sinaimg.cn/large/006tNc79gy1g46xo5yucoj30u80eg3ze.jpg)

子进程终止时会向父进程发送 CHLD 信号

TERM、INT 立刻停止  nginx 进程

QUIT 优雅的停止 nginx 进程 （处理完当前的请求在停止进程）

HUP 重载配置文件

USR1 重新打开日志文件，做日志的切割

USR2 和 WINCH 只能通过 kill 发送 做热部署时使用

一般不直接对 worker 进程发送信号

nginx 在运行时，默认会在 log/nginx.pid 文件中记录当前 master 的pid


#### nginx 的 reload 重载配置文件流程

![reload 流程1](http://ww1.sinaimg.cn/large/006tNc79gy1g46y3cgs57j30su0f0wg2.jpg)

03 可能在新的配置文件中添加的新的监听端口

子进程会继承父进程所有已经打开的监听端口

06 新的连接只会进入新的 worker 进程

![reload 流程2](http://ww3.sinaimg.cn/large/006tNc79gy1g46y4defylj30ty0eeq3n.jpg)

#### nginx 热升级流程

![热升级流程1](http://ww3.sinaimg.cn/large/006tNc79gy1g46yksz1smj30pq0eu0u7.jpg)


![热升级流程2](http://ww4.sinaimg.cn/large/006tNc79gy1g46ylbe34yj30v20e874v.jpg)












*********

并发送和 QPS tps？

QPS = 并发数/平均响应时间

并发数 = QPS*平均响应时间

cpu使用率和cpu负载

server_name 和 location 的作用：

server_name 配置不同的域名    不同域名不同虚拟主机

location    同一域名下   分发到不同的路径   或者项目 

> RPS = requests/second 每秒处理的请求数量
> concurrent connections = 并发连接数
> 在高并发的场景想往往会导致吞吐量(rps)下降

连接和请求的关系：

+ 连接是指 tcp 连接，可以在连接上进行双向的消息发送和接收。
+ 请求是指 http 请求，一个 http 请求对应一个 http 响应，这是一个完整的请求，一个连接可以对应多个 http 请求，这叫做http协议中的keepalive长连接。

并发数 = 并发连接数 = 同时存在的TCP连接数量，并发数只消耗内存，所以单纯看并发数的话，只用每个连接消耗更小的内存即可。


