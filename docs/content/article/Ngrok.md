## 手把手教你搭建ngrok服务器，实现内网穿透

ngrok是一个反向代理的小工具，可以直接从官网下载ngrok客户端，连接官方的服务，达到内网穿透目的，但由于官方服务器在国外，加上一些不可描述的原因，导致直接使用官方提供的服务比较慢。国内也有很多内网穿透的服务，如：natapp，但是追根溯源使用的都是使用该公司开放的1.7版本的源码编译的。
以下将要演示的也是用1.7版的源码进行编译，在此感谢开源世界，由此才能站在巨人的肩膀。

### 准备工作

* 可以访问公网的Linux服务器（这里选用Centos 7）
* 域名（必须已经备案）
* https证书（可选，如果没有就自己制作，但是无法真正代理https）

### 安装步骤
1. 安装git（非必须，安装是为了方便下载ngrok源码） 
```shell
yum install git
```
2. 安装golang （ngrok是由go语言开发）
```shell
yum install golang
```
3. 安装openssl (非必须，制作证书时使用）
```shell
yum install openssl
```
4. 下载源码到 `/usr/local/ngrok` 目录
```shell
git clone https://github.com/inconshreveable/ngrok.git /usr/local/ngrok
```
5. 制作证书（非必须，如果有可用的https证书就无需制作）
```shell
#这里替换为自己的域名
export NGROK_DOMAIN="yuhaowin.com"
#进入到ngrok目录生成证书
cd /usr/local/ngrok	
#下面的命令用于生成证书
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
openssl genrsa -out device.key 2048
openssl req -new -key device.key -subj "/CN=$NGROK_DOMAIN" -out device.csr
openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000
#将我们生成的证书替换ngrok默认的证书
cp rootCA.pem assets/client/tls/ngrokroot.crt
cp device.crt assets/server/tls/snakeoil.crt
cp device.key assets/server/tls/snakeoil.key
```
6. 编译服务端和客户端到 `ngrok/bin` 下
```shell
#编译64位linux服务端
GOOS=linux   GOARCH=amd64 make release-server
#编译64位windows服务端
GOOS=windows GOARCH=amd64 make release-server
#编译64位mac客户端
GOOS=darwin  GOARCH=amd64 make release-client
#编译64位windows客户端
GOOS=windows GOARCH=amd64 make release-client
```
7. 启动服务端
```shell
#指定我们刚才设置的域名，指定http, https, tcp端口号，端口号不要跟其他程序冲突
./bin/ngrokd -domain="yuhaowin.com" -httpAddr=":80" -httpsAddr=":443" -tunnelAddr=":4443"
```
8. 启动客户端
```shell
#先编写ngrok.cfg的配置文件，内容如下
server_addr: "yuhaowin.com:4443" //填写在制作证书是配置的域名，端口填写启动服务端tunnel端口，这里是4443，
trust_host_root_certs: false //如果有可用证书 true 自己制作证书 false
#Windows启动
ngrok.exe -subdomain=test -config=ngrok.cfg    8080
#Mac os启动
./ngrok   -subdomain=test -config=./ngrok.cfg  8080 
```

### 几点说明

1、服务端使用的三个端口说明
+ httpAddr 代理http服务使用的端口，默认使用 80 端口。
+ httpsAddr 代理https服务使用的端口，默认使用 443 端口。
+ tunnelAddr ngrok服务端与ngrok客户端通信端口，默认使用 4443 端口。

2、如果使用可用的https证书，在编译服务端和客户端时无需替换3个证书，但是在启动服务时需要指定证书路径。
```shell
./bin/ngrokd -tlsKey="/root/.acme.sh/yuhaowin.com/yuhaowin.com.key" -tlsCrt="/root/.acme.sh/yuhaowin.com/fullchain.cer" -domain="yuhaowin.com"  -httpAddr=":80" -httpsAddr=":443" -tunnelAddr=":4443"
```
3、[https证书获取脚本](https://github.com/Neilpang/acme.sh)
```shell
curl  https://get.acme.sh | sh
alias acme.sh=~/.acme.sh/acme.sh
export DP_Id="126022**"
export DP_Key="9777368c091db846d459757f42b43**"
acme.sh --issue --dns dns_dp -d yuhaowin.com -d *.yuhaowin.com
#生成的证书在 ～/.acme.sh 目录下

#使用 --installcert 命令,并指定目标位置, 然后证书文件会被copy到相应的位置
acme.sh --installcert  -d  <domain>.com   \
        --key-file   /etc/nginx/ssl/<domain>.key \
        --fullchain-file /etc/nginx/ssl/fullchain.cer \
        --reloadcmd  "service nginx force-reload"
```

注意：windows 用户不能使用 PowerShell 启动，必须使用 cmd 启动。

>80 端口 和 443 端口 这种特殊端口不能 ngrok 占据，过于浪费，同时又为了解决微信对非 80、443 端口的限制，服务端使用 nginx 对 ngrok 做了反向代理。因此，直接访问 http://test.yuhaowin.com 和 https://test.yuhaowin.com 即可代理 127.0.0.1:6001 服务。

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g978pykcccj30wi0iiq3w.jpg)


### nginx 反向代理 ngrok 配置
```shell
    # HTTPS server
    server {
        listen       443 ssl;
        server_name  *.yuhaowin.com;
        ssl_certificate      ../ssl/yuhaowin.com/fullchain.cer;
        ssl_certificate_key  ../ssl/yuhaowin.com/yuhaowin.com.key;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        location / {
            proxy_pass http://127.0.0.1:8848;
            proxy_set_header Host $host:8848;  #$host指与server_name相同
            proxy_redirect off;
            client_max_body_size 10m;
            client_body_buffer_size 128k;
            proxy_connect_timeout 90;
            proxy_read_timeout 90;
            proxy_buffer_size 4k;
            proxy_buffers 6 128k;
            proxy_busy_buffers_size 256k;
            proxy_temp_file_write_size 256k;
        }
        #解决配置反向代理后js css文件无法加载问题
        location ~ .*\.(js|css)$ {
            proxy_pass http://127.0.0.1:8848;
            proxy_set_header Host $host:8848; #$host指与server_name相同
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```
+ [Ngrok 使用 HTTPS 进行访问](https://blog.csdn.net/Kenon_Lin/article/details/81072656)
+ [Ngrok + Nginx 反向代理配置](https://www.jianshu.com/p/cd937631a88b)
+ [快速签发 Let's Encrypt 证书指南](https://www.cnblogs.com/esofar/p/9291685.html)

