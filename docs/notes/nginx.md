?> 更新中。。。

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


nginx的主要使用场景

+ 静态资源服务
+ 反向代理服务
	+ 负载均衡
	+ 缓存加速	 
+ api服务 

> 应用服务一般运行效率低并发性 qps 都很差，需要很多个应用服务做集群，这时就需要 nginx 有反向代理功能，讲用户的请求传导给应用服务。在很多应用服务组成的集群环境中，需要做到动态扩容和熔灾，此时就要求 nginx 具有负载均衡功能。同时为了减少请求的响应时间，可以讲一些在一段时间不变的动态内容通过 nginx 缓存起来，以加速访问。



nginx 的组成部分

![nginx的4个组成部分](http://ww3.sinaimg.cn/large/006tNc79gy1g40115hawrj30vw0fwdic.jpg)

+ Nginx 二进制文件相当汽车本身，提供各种功能。
+ Nginx.conf 相当于老司机，合理有序的控制 Nginx 的功能。
+ access.log 记录行驶轨迹，（记录 http 的请求和响应）
+ error.log 