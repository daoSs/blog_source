---
title: 深入理解ajax
date: 2017-9-29 10:15:49
tags: js
categories: 前端
---
不管前端怎么变，由客户端发起的与服务端通信使用最多的还是ajax。因此ajax无比重要，从刚进入前端世界时写的$.get() / $.put()，$.ajax()，到现在lizard框架中mvc中的model层获取数据的方法（实际上也是对$.ajax()进行各种封装）。
### XMLHttpRequest 对象
ajax技术的核心就是XMLHttpRequest对象（简称XHR），IE7+、FireFox、Chrom、Safari都支持原生XMLHttpRequest，直接new一个
``` javascript
var xhr = new XMLHttpRequest();
```
ie7-就没有这么简单啦，但这版本也太老了，做为新时代前端我有权利抛弃它，这篇文章就不赘述了。

### XMLHttpRequest用法
在使用XHR对象时要调用的第一个方法是open()。
``` javascript
xhr.open("get",url,false);
```
第三个参数表示是否发送异步请求，open只是启动一个请求以备发送，只有当调用send()时才会发送请求到服务器。send(方法的参数就是要做为请求主体发送的数据)。

在收到响应后，响应的数据会自动填充XHR对象的属性，相关介绍如下：
> * responseText：响应主体被返回的文本。
> * responseXML：如果响应内容类型是“text/xml”或“application/xml”，这个属性将保存相应数据的XML DOM文档。
> * status：响应的HTTP状态。
> * statusText：HTTP状态说明。
如果是同步请求的话，只需检测status，走成功或者失败方法。
``` javascript
if((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
    //成功
}else{
    //失败
}
```
如果是异步请求的时候我们必须检测XHR对象的readystate 属性该属性表示请求/响应过程的阶段。取值如下：
> * 0：未初始化。尚未调用open()。
> * 1：启动。已调用open()，未调用send()。
> * 2：发送。已调用send()，未收到响应。
> * 3：接收。接到部分响应数据。
> * 4：完成。已经接受到全部数据。
readystatus一变就会触发一次readystatechange事件：
``` javascript
xhr.onreadystatechange = function(){
    if(xhr.readyStatus == 4){
      if((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
          //成功
        }else{
          //失败
        } 
    }
}
```
默认情况下XHR还会发送HTTP头部信息：
> * Accept：浏览器能处理的内容
> * Accept-Charset：浏览器能显示的字符集
> * Accept-Encoding：浏览器能处理的压缩编码
> * Accept-Language：浏览器当前设置的语言
> * connection：浏览器与服务器之间连接的类型
> * cookie：当前页面设置的任何cookie
> * Host：请求所在域
> * Referer：发出页面请求的URI
> * User-Agent：浏览器用户代理字符串
使用setRequestHeader()方法可以设置自定义请求头部信息。头部字段名称和头部字段值做为key，value形式作为参数。用户也可以自定义头部信息，这样在服务器收到请求后可以进行后续操作。

同样的，调用XHR对象的getResponseHeader()传入头部字段名称，或者调用getAllResponseHeaders()可以获取头部信息的字符串。

因此，服务器端返回给客户端的response可以增加额外的、结构化的数据供前端使用。
### GET 、POST
get、post请求大家都太熟悉了，大家都知道从请求参数来看get是把参数附在url后面，post是把请求参数作为请求的主体提交。但是成熟的ajax方法内部是如何去实现的呢。以jQuery、zepto.js封装好的api请求参数来看:
``` javascript
url =   options.url || "", //请求的链接
        type = (options.type || "get").toLowerCase(), //请求的方法,默认为get
        data = options.data || null, //请求的数据
        contentType = options.contentType || "", //请求头
        dataType = options.dataType || "", //请求的类型
        async = options.async === undefined && true, //是否异步，默认为true.
        timeOut = options.timeOut, //超时时间。 
        before = options.before || function(){}, //发送之前执行的函数
        error = options.error || function(){}, //错误执行的函数
        success = options.success || function() {}; //请求成功的回调函数
```
使用者只需要将type值设置为get或post就可指定这次请求是用post方式还是get方式请求。

可是实际上，get请求时，XHR对象的send()方法的参数需要是null，需要我们手动把参数附在url后，如下：
``` javascript
function addUrlParam(url,name,value){
    url += (url.indexOf("?") == -1 ? "?" : "&");
    url += encodeURIComponent(name) + "=" + encodeURIComponent(value);
}
```
对post请求而言，直接将data格式化一下塞到send的参数中即可
``` javascript
if (data) {
        if (typeof data === "string") {
            data = data.split("&");
            for (var i = 0, len = data.length; i < len; i++) {
                name = data[i].split("=")[0];
                value = data[i].split("=")[1];
                data[i] =  encodeURIComponent(name) + "=" + encodeURIComponent(value);
            }
            data = data.replace("/%20/g", "+");
        } else if (typeof data === "object") {
            var arr = [];
            for (var name in data) {
                var value = data[name].toString();
                name = encodeURIComponent(name);
                value = encodeURIComponent(value);
                arr.push(name + "=" + value);
            }
            data = arr.join("&").replace("/%20/g", "+");
        }
}
```
另外对post请求来说，因为XHR设计的目的是处理XML，为了服务器端解析时对post发送的数据和提交web表单的请求一视同仁，我们首先需要调用setRequestHeader方法把Content-Type 头部信息设置为application/x-www-form-urlencoded
``` javascript
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
```
并且由前端设置请求字符串，当然字符串也要经过前端的处理。

到这里，我们基本上可以实现一个简单的ajax方法，但是如果要运用到实际生产中，这个方法是不健壮的，会有诸多问题。本文标题深入理解ajax，所以还要继续深入下去，逐步完善我们的ajax方法。

### 超时设定
一个健壮的ajax方法一定是要用户可以自由设定超时时间的。在XMLHttpRequest对象在早期是没有自己的超时方法的，后来W3C推出了XMLHttpRequest 2级规范，向其中添加了格式化数据（formData()），超时设定（timeout）等方法。

经过formData的append方法添加的健值对是可以直接通过send方法发送的，在这里不再赘述。

如果当前浏览器的XHR对象支持timeout的话，我们可以这样设置超时时间：
``` javascript 
xhr.onreadystatechange = function(){
    if(xhr.readyStatus == 4){
      try{
       if((xhr.status >= 200 && xhr.status < 300) || xhr.status ==304){
             //成功
           }else{
             //失败
          } 
        } catch(ex) {
             由ontimeout事件处理程序处理
          }
      
    }
}
xhr.timeout = 1000;//单位是毫秒
```
当然，浏览器稍微老一点我们就得麻烦点，自己用settimeout写个定时器，时间到了以后标志位改变并且调用xhr.abort()方法取消异步请求。
``` javascript
function setTime(callback, script) {
        if (timeOut !== undefined) {
            timeout_flag = setTimeout(function() {
                
                    timeout_bool = true;
                    xhr && xhr.abort();
                
                console.log("timeout");

            }, timeOut);
        }
    }

xhr.onreadystatechange = function() {
            if (xhr.readyState === 4) {
                if (timeOut !== undefined) {
                    //由于执行abort()方法后，有可能触发onreadystatechange事件，
                    //所以设置一个timeout_bool标识，来忽略中止触发的事件。
                    if (timeout_bool) {
                        return;
                    }
                    clearTimeout(timeout_flag);
                }
                if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304) {

                    success(xhr.responseText);
                } else {
                     error(xhr.status, xhr.statusText);
                }
            }
        };
```
### 跨域

跨域安全策略是对由XHR实现的ajax通信的一个主要限制，但是多数情况下我们必须要访问不同域的资源。CORS是w3c定义的关于必须访问跨域资源时，浏览器与服务器应该如何沟通。对于get post请求来说，发送请求时需要附加一个额外的Origin头部，其中包含请求的源信息（协议、域名和端口），以便服务器端根据这个头部信息来决定是否响应。如果服务器端认为这个请求可以接受，那么就会在Access-Control-Allow-Origin投不中回发相同的源信息（可以回发“*”）,如果没有这个头或者有这个头但不匹配的话浏览器会驳回请求。

跨域施就算成功请求也是会有一些限制：
> * 不能使用setRequestHeader()设置自定义头
> * 不能发送和接受cookie
> * 调用getAllRequestHeaders()方法会返回空字符串

在CORS之前，出现过一些处理跨域的奇淫技巧，，比如说之前公司（15年）还在使用的jsonp，具体原理是利用script标签没有跨域问题这个特性（准确说是所有带src属性的标签都有这个特性）。
