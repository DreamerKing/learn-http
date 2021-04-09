# HTTP/0.9
只有GET方法,没有首部。设计目的是获取HTML(即文本,没有图片)

# HTTP/1.0 
+ 首部
+ 响应码
+ 重定向
+ 错误
+ 条件请求
+ 内容编码(压缩)
+ 更多的请求方法
  ...
问题：
不能让多个请求共用一个连接,缺少强制的Host首部,并且缓存选择也相当简陋。

# HTTP/1.1 
修复了HTTP/1.0中的大量问题。强制要求客户端提供Host首部,让虚拟主机服务托管成为可能，也就是在一个IP上提供多个Web服务。当使用新连接指令时，Web服务器也不需要在每个响应之后关闭连接。(连接复用)
+ 缓存相关首部扩展
+ OPTIONS方法
+ Upgrade首部
+ Range(范围)请求
+ 压缩和传输编码(transfer-encoding)
+ 管道化(pipelining) 允许客户端次发起多个请求 存在队头阻塞问题
  
  H1的问题
  1. 队头阻塞
  2. 低效的TCP利用
  3. 臃肿的消息首部
  4. 受限的优先级设置
  5. 第三方资源

# SPDY 
一种HTTP替代协议,为HTTP/2奠定了基础
+ 多路复用
+ 帧和首部压缩

# HTTP/2.0
+ 相比于使用TCP的HTTP/1.1,最终用户可感知的多数延迟都有能够量化的显著改善;
+ 解HTTP中的队首阻塞问题
+ 并行实现机制不依赖与服务器建立多个连接，从而提升TCP连接的利用率
+ 保留HTTP的语义,可以利用已有文档资源,包括(但不限于)HTTP方法、状态码、URL和首部字段
+ 明确定义HTTP/2.0与HTTP/1.x交互方法,特别是通过中介的方法(双向)
+ 明确提出可以被合理使用的新的扩展点和策略

# H2服务搭建
+ 获取并安装支持H2的Web服务
+ 下载并安装TLS证书,让浏览器与服务器可以通过h2连接

获取证书的方式
+ 使用在线资源 自签名证书服务 [https 自签名SSL证书](https://www.cnblogs.com/aaron-agu/p/10560659.html)
+ 自己创建证书 openssl
  ```
  openssl genrsa -out key.pem 2048
  openssl req -new -x509 -sha256 -key key.pem -out cert.pem -days 365  -subj "/CN=dk.com"
  ```
+ 向数字证书认证机构申请 Let Encrypt
  ```sh
  wget https://dl.eff.org/certbot-auto
  chomd a+x certbot-auto
  ./cerbot-auto certonly --webroot -w <your web root> -d <your domain>
  ```

HTTP/2服务器
[Nghttp2](http://nghttp2.org/)
```sh
sudo apt-get install nghttp2
nghttpd -v -d ./ 8989 ./key.pem ./cert.pem
```

# Web优化
## 剖析页面请求
大致过程分成两部分: 资源获取和页面解析渲染
资源请求获取流程：
1. 将请求URL放入请求队列
2. 解析URL中的域名IP地址
3. 建立与目标主机的TCP连接
4. 如果是HTTPS请求,初始化并完成TSL握手
5. 向页面对应URL的发送请求
   
接收响应及页面渲流程
1. 接收响应
2. 如果接收的是HTML主体，那么解析它,并对页面中的资源触发优先获取机制
3. 如果页面的关键资源已接收到,就开始渲染页面
4. 接收其他资源,继续解析页面,直到结束

## 关键性指标
1. 延迟 RTT(往返延迟) 主要瓶颈 IP数据包在网络上所花费的时间
2. 带宽 可能成为性能瓶颈
3. DNS查询 域名转IP
4. 建立连接时间(三次握手)
5. TLS协商时间
6. 首字节时间(TTFB)
7. 内容下载时间
8. 开始渲染时间
9. 文档(页面)加载时间

现状：
+ 更多的字节
+ 更多的资源
+ 更高的复杂度
+ 更多的域名
+ 更多的TCP socket 

