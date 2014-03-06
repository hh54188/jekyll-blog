## The life of a resource request on the wire

> **Cache**: Given the URL of a resource on the web, the browser starts by checking its local and application caches. If you have previously fetched the resource and the appropriate cache headers were provided (Expires, Cache-Control, etc.), then it is possible that we may be allowed to use the local copy to fulfill the request - **the fastest request is a request not made.**

|

> **Socket Reuse**: Given a hostname and resource path, Chrome first checks for existing open connections it is allowed to reuse - sockets are pooled by {scheme, host, port}.

|

> **DNS**: Finally, if neither of the above conditions is matched, then the request must begin by resolving the hostname to its IP address - aka, a DNS lookup.

|

> **TCP**: With the resolved IP address in hand, Chrome can now open a new TCP connection to the destination, which means that we must perform the "three-way handshake"

|

> **SSL**: Once the TCP handshake is complete, and if we're connecting to a secure destination (HTTPS), then the SSL handshake must take place.

## TCP

> A TCP connection provides a **full-duplex service**: If there is a TCP connection
between Process A on one host and Process B on another host, then applicationlayer data can flow from Process A to Process B at the same time as application layer data flows from Process B to Process A

### How End

> Either of the two processes participating in a TCP connection can **end** the connection. When a connection ends, the “resources” (that is, the buffers and variables)
in the hosts are deallocated.

|

>**Suppose the client decides to close the
connection**, as shown in Figure 3.40. The client application process issues a close command. This causes the client TCP to send a special TCP segment to the server process. This special segment has a flag bit in the segment’s header, the FIN bit set to 1

|

>When the server receives this segment, it sends the client an acknowledgment segment in return. The server then sends its own shutdown segment, which has the FIN bit set to 1.

|

>Finally, the client acknowledges the server’s shutdown segment. At this point, all the resources in the two hosts are now deallocated.

## DNS

>1. The same user machine runs the client side of the DNS application.
2. The browser extracts the hostname, www.someschool.edu, from the URL
and passes the hostname to the client side of the DNS application.
3. The DNS client sends a query containing the hostname to a DNS server.
4. The DNS client eventually receives a reply, which includes the IP address for
the hostname.
5. Once the browser receives the IP address from DNS, it can initiate a TCP connection to the HTTP server process located at port 80 at that IP address.

|

>The DNS protocol runs over UDP and uses port 53.

![dns_root_TLD](./images/dns_root_TLD.png)

![dns_query](./images/dns_query.png)

### HTTP

>HTTP uses TCP as its underlying transport protocol (rather than running on top of
UDP). The HTTP client first initiates a TCP connection with the server. Once the connection is established, the browser and the server processes access TCP through their socket interfaces. As described in Section 2.1, on the client side the socket interface is the door between the client process and the TCP connection; on the server side it is the door between the server process and the TCP connection. The client sends HTTP request messages into its socket interface and receives HTTP response messages from its socket interface.

|

>Once the client sends a message into its socket interface, the message is out of the client’s hands and is
“in the hands” of TCP.

|

> Here we see one of the great advantages of a layered architecture—HTTP need not worry about lost data or the
details of how TCP recovers from loss or reordering of data within the network. That is the job of TCP and the protocols in the lower layers of the protocol stack.

### Network Legend 

![resource network timeline](./images/resource-timing-overview.png)

![legend](./images/network_legend.png)


[1 external script in head, 3 before closing body tag](http://stevesouders.com/cuzillion/?c0=hc1hfff2_0_f&c1=hj1hfff2_0_f&c2=bi1hfff2_0_f&c3=bi1hfff2_0_f&c4=bi1hfff2_0_f&c5=bi1hfff2_0_f&c6=bi1hfff2_0_f&c7=bi1hfff2_0_f&c8=bi1hfff2_0_f&c9=bi1hfff2_0_f&c10=bj1hfff2_0_f&c11=bj1hfff2_0_f&c12=bj1hfff2_0_f&t=1312331291063)

**Chrome**
![chrome](./images/1-script-head-3-script-body-chrome.png)
**Firefox**
![firefox](./images/1-script-head-3-script-body-firefox.png)
**IE7**
![ie7](./images/1-script-head-3-script-body-ie7.png)
**IE8**
![ie8](./images/1-script-head-3-script-body-ie8.png)

Chrome/IE/Firefox中，如果head中有script标签（无论是否带有aync），会阻塞body的解析

但是在Firefox中，底部的script不会被提前，在IE中，底部script会提前至head进行解析，会阻塞body的解析

无论头部有没有script标签（仅有link），在IE和firefox中，head的解析会阻塞body的解析：

http://stevesouders.com/cuzillion/?c0=hc1hfff2_0_f&c1=hc1hfff2_0_f&c2=bi1hfff2_0_f&c3=bi1hfff2_0_f&c4=bi1hfff2_0_f&c5=bi1hfff2_0_f&c6=bi1hfff2_0_f&c7=bi1hfff2_0_f&c8=bi1hfff2_0_f&c9=bi1hfff2_0_f&c10=bj1hfff2_0_f&c11=bj1hfff2_0_f&c12=bj1hfff2_0_f&t=1393989334360


[1 inline script in head, 4 before closing body tag](http://stevesouders.com/cuzillion/?c0=hc1hfff2_0_f&c1=hb0hfff0_0_f&c2=bi1hfff2_0_f&c3=bi1hfff2_0_f&c4=bi1hfff2_0_f&c5=bi1hfff2_0_f&c6=bi1hfff2_0_f&c7=bi1hfff2_0_f&c8=bi1hfff2_0_f&c9=bi1hfff2_0_f&c10=bj1hfff2_0_f&c11=bj1hfff2_0_f&c12=bj1hfff2_0_f&c13=bj1hfff2_0_f&t=1393213090671)


**Chrome**
![chrome](./images/1-inline-script-head-4-script-body-chrome.png)
**Firefox**
![firefox](./images/1-inline-script-head-4-script-body-firefox.png)
**IE8**
![ie7](./images/1-inline-script-head-4-script-body-ie8.png)



[All scripts in bottom](http://stevesouders.com/cuzillion/?c0=hc1hfff2_0_f&c1=bi1hfff2_0_f&c2=bi1hfff2_0_f&c3=bi1hfff2_0_f&c4=bi1hfff2_0_f&c5=bi1hfff2_0_f&c6=bi1hfff2_0_f&c7=bi1hfff2_0_f&c8=bi1hfff2_0_f&c9=bj1hfff2_0_f&c10=bj1hfff2_0_f&c11=bj1hfff2_0_f&c12=bj1hfff2_0_f&t=1312335422)

**Chrome**
![chrome](./images/all-script-in-bottom-chrome.png)
**Firefox**
![firefox](./images/all-script-in-bottom-firefox.png)
**IE7**
![ie7](./images/all-script-in-bottom-ie7.png)
**IE8**
![ie8](./images/all-script-in-bottom-ie8.png)




---

http://blogs.msdn.com/b/ieinternals/archive/2011/07/18/optimal-html-head-ordering-to-avoid-parser-restarts-redownloads-and-improve-performance.aspx:

To mitigate this issue when loading a page, Internet Explorer runs a second instance of a parser whose job is to hunt for resources to download while the main parser is paused. This mode is called the lookahead pre-parser[3] because it looks ahead of the main parser for resources referenced in later markup. The download requests triggered by the lookahead are called “speculative” because it is possible (not likely, but possible) that the script run by the main parser will change the meaning of the subsequent markup (for instance, it might adjust the BASE against which relative URLs are combined) and result in the speculative request being wasted.

http://www.ravelrumba.com/blog/script-downloading-chrome/:

- An external JS file in the head blocks the parser. This kicks off the PreloadScanner, which attempts to download all other scripts and stylesheets — but not images.

- Preloaded scripts do not block images and other assets.

- The main parser downloads each resource as it encounters them (exactly how you’d expect). The PreloadScanner has some subtleties. For instance, it will only download scripts and stylesheets while the parser is blocked in the head. This is designed so that images (which will never block the main parser) will not contend for bandwidth with the critical resources that can block the parser. This allows the page to display more progressively with images filling in around the content.

It is possible to contrive test cases where this leads to inefficiencies. There’s a good example here: https://bugs.webkit.org/show_bug.cgi?id=45072



很奇怪，图片不会被预加载，而且如果脚本预加载的话，body会被阻塞（但是script和link会被提前）

http://stevesouders.com/cuzillion/?c0=hj1hfff2_0_f&c1=bi1hfff2_0_f&c2=bi1hfff2_0_f&c3=bi1hfff2_0_f&c4=bc1hfff2_0_f&c5=bj1hfff2_0_f&t=1393167087614

头部script，body三个image，一个link，一个script

---

# 利用浏览器特性对资源加载进行优化

## 过去：Preloader

preloader并不是它唯一的名称，至少在Chrome中可以这么称呼它，或者preload scanner；在IE中可以称之为Lookahead Pre-parser。

直至今天，据统计Alexa中排名前一百万的网站，[载入时间的百分之69.5的时间仍然都花在网络的响应(network latency)上](https://docs.google.com/presentation/d/1omz7uv3CLR4fE1kRbVl1_t55XLcVXtW1pAV0-HoW6xQ/present#slide=id.g33a803cd_4_320)；并且当你遇上了一些没有defer或asyc属性的外链脚本时，为了执行它们还不得不暂停页面的解析和渲染。preloader便是在这么一个场景下产生的，即便是页面暂停了解析和渲染，浏览器还是会尽可能的去页面上寻找可能需要被加载的资源（外链样式，外链脚本或者图片；不包括iframe，video）去加载。这样能够缩短页面等待网络响应的时间。当然前提是你的浏览器必须支持页面的解析与下载并行。

IE8除了将每台host的最高并行下载的资源数从2个提升到6个之外，最大的提升莫过于允许并行下载脚本文件了。

比如下面这个[页面](http://stevesouders.com/cuzillion/?c0=hc1hfff2_0_f&c1=hc1hfff2_0_f&c2=hj1hfff2_0_f&c3=hj1hfff2_0_f&c4=bi1hfff2_0_f&c5=bi1hfff2_0_f&c6=bj1hfff2_0_f&c7=bi1hfff2_0_f&t=1382383139903)

```
//只是大意表示资源分部情况
<head>
    <link rel="stylesheet" type="text/css" href="">
    <link rel="stylesheet" type="text/css" href="">
    <script type="text/javascript"></script>
    <script type="text/javascript"></script>
</head>
<body>
    <img src="">
    <img src="">
    <script type="text/javascript"></script>
    <img src="">
</body>
```

在IE7下网络请求的瀑布图如下图所示：

![no-pre-loader-waterfall-ie7](./images/no-pre-loader-waterfall-ie7.png)

我们能看到head中的外链样式能够并行下载，body中的图片能并行下载，但惟独外链脚本不行。

但如果你在IE8中查看网络请求的，瀑布图如下图所示

![no-pre-loader-waterfall-ie7](./images/pre-loader-waterfall-ie8.png)

head中的脚本和外链样式已经可以并行下载了。并且我们可以看到整个页面的载入时间从14s下降到7s。

大致从08年开始Internet Explorer, WebKit和Mozilla都开始使用preload这么一项技术。与浏览器引擎类似，虽然实现的[原理类似](http://www.html5rocks.com/en/tutorials/internals/howbrowserswork/#Main_flow_examples)，但是还有细节上的差别。

在上面的例子中，虽然在IE8每台主机最高并行下载数提高到了6，页面上的所有资源也都属于同一个域下(domain1)，但第一阶段很明显只有head中的四个资源在并行下载，并且阻塞了body里的资源下载。因为在IE中，head中的pre-parser是不会“跨”入body中去加载资源的。

但是其他的浏览器并非如此，比如Chrome，比如这个[页面](http://stevesouders.com/cuzillion/?c0=hc1hfff2_0_f&c1=hj1hfff2_0_f&c2=bi1hfff2_0_f&c3=bi1hfff2_0_f&c4=bi1hfff2_0_f&c5=bi1hfff2_0_f&c6=bi1hfff2_0_f&c7=bi1hfff2_0_f&c8=bi1hfff2_0_f&c9=bi1hfff2_0_f&c10=bj1hfff2_0_f&c11=bj1hfff2_0_f&c12=bj1hfff2_0_f&t=1312331291063)

```
<head>
    <link rel="stylesheet" type="text/css" href="">
    <script type="text/javascript"></script>
</head>
<body>
    <img src="">
    <img src="">
    <img src="">
    <img src="">
    <img src="">
    <img src="">
    <img src="">
    <img src="">
    <script type="text/javascript"></script>
    <script type="text/javascript"></script>
    <script type="text/javascript"></script>
</body>
```

我们首先来看看它加载的瀑布图：

![chrome-waterfall](./images/chrome-waterfall.jpg)

虽然有三个外链脚本在body内，在页面的底部，但是Chrome还是选择优先加载外链脚本。preloader通常是在遇见第一个



