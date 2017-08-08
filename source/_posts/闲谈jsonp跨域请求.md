---
title: 闲谈jquery中jsonp跨域请求
date: 2017-07-25 16:10:19
tags: js
---
## 介绍
JSON数据是一种能很方便通过js解析的结构化数据。如果获取的数据文件存放在远程服务器上(域名不同,也就是跨域数据),则需要使用jsonp类型，使用这种类型的话，会创建一个查询字符串参数 callback=? ，这个参数会加在请求的URL后面。**服务器端应当在JSON数据前加上回调函数名，以便完成一个有效的JSONP请求。** 如果要指定回调函数的参数名来取代默认的callback，可以通过设置$.ajax()的jsonp参数。

其实jquery跨域的原理是通过外链  ` <script> ` 来实现的,然后在通过回调函数加上回调函数的参数来实现真正的跨域。

Jquery 在每次跨域发送请求时都会有callback这个参数，其实这个参数的值就是回调函数名称， **所以，服务器端在发送json数据时，应该把这个参数放到前面，** 这个参数的值往往是随机生成的，如：jsonp1294734708682，同时也可以通过 $.ajax 方法设置 callback 方法的名称。明白了原理后，服务器端应该这样发送数据：

 <pre><code>string message = "jsonp1294734708682({\"userid\":0,\"username\":\"null\"})";</code></pre>

这样，json 数据 {\"userid\":0,\"username\":\"null\"} 就作为了 jsonp1294734708682 回调函数的一个参数。

## 实例

<pre><code>$.ajax({
    url: 'xxxx',
    dataType: "jsonp",
    jsonp: "callbackparam",//传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(默认为:callback)
    jsonpCallback:"success_jsonpCallback",//自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名
    success: function (data) {
        $.each(data.success, function (i, n) {
            $("#msg").append( n.title);
        });
    }
});</code></pre>

服务器需要这样返回数据示例:

<pre><code><%@ WebHandler Language="C#" Class="ajax" %>
using System;
 using System.Web;
public class ajax : IHttpHandler {    
   public void ProcessRequest (HttpContext context) {
          context.Response.ContentType = "text/plain";
         string callbackFunName = context.Request["callbackparam"];
         context.Response.Write(success_jsonpCallback + "([ { name:\"John\"} ] )");
     }
       public bool IsReusable {
         get {             return false;       
           }
     }

 }</code></pre>

 ![言sir](http://pic002.cnblogs.com/images/2011/75158/2011100917363685.gif)

  ajax jsonp与普通的ajax请求的主要区别在于——请求响应结果的处理。如上面代码所示的响应结果为：
  success_jsonpCallback([ { name:"John"} ] );

  其实就是，调用jsonp回调函数success_jsonpCallback,并将要响应的字符串或json传入此方法(作为参数值)，
  其底层的实现，大概的猜想应该是：
    success_jsonpCallback([ { name:"John"} ] );
  function success_jsonpCallback(data)
  {
        success(data);
  }

  ![言sir](http://oqsmnfdij.bkt.clouddn.com/images/pay.png)

  <p style="text-align: center;display: block;
      width: 100px;
      margin:0 auto;
      height: 30px;
      line-height: 30px;
      background-color: #E74851;
      color: #fff;
      text-align: center;
      text-decoration: none;
      border-radius: 8px;
      font-weight: bold;
      font-size: 16px;">打赏一个呗</p>
