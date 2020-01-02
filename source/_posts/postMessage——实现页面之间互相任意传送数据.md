---
title: postMessage——实现页面之间互相任意传送数据
date: 2019-03-26 
categories: 
- web前端
tags:
- 跨域
- postMessage
- 页面间消息传送
---
参考文章：https://www.jb51.net/html5/471062.html

很早之前看跨域资料时，就想到页面之间互相任意传送数据的可能性，今天根据链接中的文章实现了。

首先，需要开设两个服务端口（为了显示即便跨域也是可以的才用两个，也可以开设一个端口）。如果用webstrom自带一个，就另外架设一个服务就行。

其次，实现原理上；
<!--more-->
1、a页面，用open方法打开b页面（获取到b页面的window对象，就可以实现向b页面传送数据）。

2、b页面，添加onmessage事件，就可以接收到a页面的window对象，并进而向a页面传送数据

最后是代码：

1、a页面代码：

// postMessage实验，本页面用端口服务5000打开，加载后会自动打开testFilfer.html页面（在端口63342）

var domain = http://localhost:63342;

var myPopup = window.open(domain/linkTomysql/testFilter.html;myWindow;);

//周期性的发送消息

setInterval(function(){

    var message = Hello!  The time is:  + (new Date().getTime());

    console.log(blog.local:  sending message:   + message);

    //send the message and target URI

    myPopup.postMessage(message,domain);

},6000);

//监听消息反馈，接收testFilter页面的数据

window.addEventListener(message,function(event) {

    if(event.origin !== http://localhost:63342) return;

    console.log(received response:,event.data);

},false);

2、b页面代码：


    // 接收test-grid-css的数据

    window.addEventListener(message,function(event) {

        if(event.origin !== http://localhost:5000) return;

        console.log(message received:   + event.data,event);

        event.source.postMessage(holla back youngin,event.origin);

    },false);
