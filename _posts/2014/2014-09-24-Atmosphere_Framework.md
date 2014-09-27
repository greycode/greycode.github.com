---
layout: post
title: Atmosphere Framework
categories:
- Java开发
tags:
- [java,websocket,comet,长轮询,服务器推送,framework]
---
<link rel="stylesheet" href="/media/highlight/styles/github.css">
<script src="/media/highlight/highlight.pack.js"></script>
<script>
$(document).ready(function() {
  $('pre code').each(function(i, block) {
    hljs.highlightBlock(block);
  });
});
</script>
<img src="/media/pic2014/Atmosphere-logo.png" alt="">
###What is Atmosphere
Atmosphere 是一个可用于开发企业级异步Java程序的框架，对服务器与客户端的双向通讯技术（WebSocket、Server Side Events以及传统的Ajax技术如长轮询（long polling）等）进行了一层封装，从而可以开发各种实时异步的应用，比如仪表板，聊天室，实时报价等，[支持主流的服务器与浏览器](https://github.com/Atmosphere/atmosphere/wiki/Supported-WebServers-and-Browsers)。
###Why Atmosphere
对于开发实时或异步应用，现行的解决方案大概有以下几种：  
　　1.长轮询（Long Polling）  
　　2.HTTP流（Http Streaming）/[Comet](http://en.wikipedia.org/wiki/Comet_(programming))  
　　3.[服务器发送事件（Server Sent Events）](https://developer.mozilla.org/zh-CN/docs/Server-sent_events)  
　　4.JSONP  
　　5.[Websocket](http://zh.wikipedia.org/wiki/WebSocket)  
相比较（如[Websocket对比SSE、LP](http://www.oschina.net/question/82993_63312)和[Comet对比Websocket](http://www.infoq.com/cn/news/2008/12/websockets-vs-comet-ajax)）而言，Websocket更加标准、性能好，但是出现晚，浏览器和应用服务器（代理服务器）的支持力度不同。其他技术虽有弊端却受更多的支持（微信的网页端貌似用的就是Long Polling）。
Atmosphere所解决的问题，就是在通信的两端提供了一层抽象，隐藏了低端的实现，让我们在开发时更多的专注于业务逻辑而不是浏览器或服务器的通讯技术，若服务器/浏览器支持Websocket协议，则使用Websocket，否则将自动回退（fallback）到其他如Long Polling（[怎样使用Atmosphere检测浏览器服务器对上述各种技术是否支持](https://github.com/Atmosphere/atmosphere/wiki/How-to-discover-supported-transport-between-browsers-and-server)）。  
另外该框架([主页](http://async-io.org/))：  
　　·在Reverb, Wall Street Journal, GameDuell, VMWare, Atlassian等都有使用。  
　　·易于扩展（scale）  
　　·支持云/集群  
Atmosphere可以做到两端（包括Javascript库哦）通吃，其开发的应用完全可以部署到现各种流行的应用服务器如WebLogic, Tomcat, Jetty, GlassFish, Vert.x, Netty Framework 等等。
<img src="/media/pic2014/Atmosphere -framework.png" alt="">
###How to Use
最详细的说明请参考框架的[WIKI](https://github.com/Atmosphere/atmosphere/wiki)与[Samples](https://github.com/Atmosphere/atmosphere-samples)。

Server
<pre><code class="java">
import org.atmosphere.config.service.ManagedService;
import org.atmosphere.cpr.AtmosphereResponse;
import org.atmosphere.handler.OnMessage;

@ManagedService(path = "/echo")
public class Echo {
    @onMessage
    public void onMessage(AtmosphereResponse res, String m) {
        res.write("Echo: " + m);
    }
}
</code></pre>
Client
<pre><code class="javascript">
$(function () {
   var request = {
     url: document.location.toString() + 'echo',
     transport : "websocket" ,
     fallbackTransport: 'long-polling'};

   request.onMessage = function (response) {
     console.log(response.responseBody)
   };
   $.atmosphere.subscribe(request).push("Hello");
}
</code></pre>
上面是官网给出的样例代码，我的Glist上有个[简单的聊天应用](https://gist.github.com/greycode/088587fc6b55b2e8a4d3)，可以参考下，同时建议部署到Tomcat 8上测试。

###More Resource
1.[Spring 4 已经支持](http://www.oschina.net/translate/websocket-architecture-in-spring-4-0)Websocket。
