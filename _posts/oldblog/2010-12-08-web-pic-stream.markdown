---
layout: post
title: "“ClientAbortException: java.net.SocketException:”异常的问题"
date: 2013-07-26 22:35:00
---

前几天，在做图片以stream形式输出到页面上展现的时候，后台一直报异常，且页面上图片无法正常现实。异常内容为：  
{% highlight java %}
ClientAbortException:  java.net.SocketException: Software caused connection abort: socket write error  
at org.apache.catalina.connector.OutputBuffer.realWriteBytes(OutputBuffer.java:358)  
at org.apache.tomcat.util.buf.ByteChunk.flushBuffer(ByteChunk.java:434)  
at org.apache.catalina.connector.OutputBuffer.doFlush(OutputBuffer.java:309)  
at org.apache.catalina.connector.OutputBuffer.flush(OutputBuffer.java:288)    
at org.apache.catalina.connector.CoyoteOutputStream.flush(CoyoteOutputStream.java:98)  
at javax.imageio.stream.FileCacheImageOutputStream.close(FileCacheImageOutputStream.java:213)  
......  
{% endhighlight %}

在网上找了好久，发现是个普遍问题，但原因各异，有人总结为：  
1：服务器的并发连接数超过了其承载量，服务器会将其中一些连接Down掉；  
2：客户关掉了浏览器，而服务器还在给客户端发送数据；  
3：浏览器端按了Stop；  
4：服务器给客户端响应结果给防火墙拦截了。  

这些原因太过笼统，细查错误很难。  
偶然在一个英文网页上看到了条有关这个异常的，本人英语水平差，只能说个大意：  
这个已知异常出现在IE浏览器显示tif格式的图片的时候。  

亲自试验了下，的确在chrome和火狐上显示时，后台不会报次错误，不过图片显示有点异样。因为图片是存于数据库的BLOB类型的字段中，所以一直没注意图片的格式，后来才发现，原来图片是gif格式，而我使用ImageIO.write(img, "jpeg", response.getOutputStream());输出成jpg格式的图片，以致导致IE浏览器对图片解析有问题。在此将jpeg改成gif后不再出现异常。  

忙乎了一天多，结果问题出在这里。  

其问题导致的原因可能是：IE浏览器在解析错误图片格式或不支持的图片格式时，可能向服务器发送了多次请求或是直接关闭了连接，从而导致了tomcat报次异常。   
