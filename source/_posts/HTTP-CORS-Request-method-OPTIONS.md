---
title: HTTP CORS Request method OPTIONS
date: 2018-04-20 11:02:46
categories:
- keep learning
tags:
- HTTP
---

# 问题描述
浏览器发出AJAX请求，后台服务端限制访问方式是POST，并通过判断content type是否为application/json作为参数类型校验，但是当加上content type之后request method变成了options,最后发现这是由于跨域（源）访问所发起的预检请求。

# 一个源的定义

同源策略限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的重要安全机制。
一个源的定义
如果两个页面的协议，端口（如果有指定）和域名都相同，则两个页面具有相同的源。

下表给出了相对http://store.company.com/dir/page.html 同源检测的示例:

	-------------------------------------------------------------------------------------
	| URL                                             | 结果  | 原因                    |
	-------------------------------------------------------------------------------------
	| http://store.company.com/dir2/other.html        | 成功  |                         |
	-------------------------------------------------------------------------------------
	| http://store.company.com/dir/inner/another.html | 成功  |                         |
	-------------------------------------------------------------------------------------
	| https://store.company.com/secure.html           | 失败  | 不同协议 ( https和http ) |
	-------------------------------------------------------------------------------------
	| http://store.company.com:81/dir/etc.html        | 失败  | 不同端口 ( 81和80)       |
	-------------------------------------------------------------------------------------
	| http://news.company.com/dir/other.html          | 失败  | 不同域名 ( news和store ) |
	-------------------------------------------------------------------------------------

<!--more-->
# 跨源网络访问
同源策略控制了不同源之间的交互，例如在使用XMLHttpRequest 或 img 标签时则会受到同源策略的约束。这些交互通常分为三类：

 - 通常允许跨域写操作（Cross-origin writes）。例如链接（links），重定向以及表单提交。** 特定少数的HTTP请求需要添加 preflight。**（预检请求,即本文开头所提到的）
 - 通常允许跨域资源嵌入（Cross-origin embedding）。之后下面会举例说明。
 - 通常不允许跨域读操作（Cross-origin reads）。但常可以通过内嵌资源来巧妙的进行读取访问。例如可以读取嵌入图片的高度和宽度，调用内嵌脚本的方法，或availability of an embedded resource.

以下是可能嵌入跨源的资源的一些示例：
```html
<script src="..."></script> 标签嵌入跨域脚本。语法错误信息只能在同源脚本中捕捉到。
<link rel="stylesheet" href="..."> 标签嵌入CSS。由于CSS的松散的语法规则，CSS的跨域需要一个设置正确的Content-Type 消息头。不同浏览器有不同的限制： IE, Firefox, Chrome, Safari (跳至CVE-2010-0051)部分 和 Opera。
<img>嵌入图片。支持的图片格式包括PNG,JPEG,GIF,BMP,SVG,...
<video> 和 <audio>嵌入多媒体资源。
<object>, <embed> 和 <applet> 的插件。
@font-face 引入的字体。一些浏览器允许跨域字体（ cross-origin fonts），一些需要同源字体（same-origin fonts）。
<frame> 和 <iframe> 载入的任何资源。站点可以使用X-Frame-Options消息头来阻止这种形式的跨域交互。
```

# 预检请求
不同于一些简单的请求不会触发 CORS 预检请求，“需预检的请求”要求必须首先使用 OPTIONS   方法发起一个预检请求到服务器，以获知服务器是否允许该实际请求。"预检请求“的使用，可以避免跨域请求对服务器的用户数据产生未预期的影响。

当请求满足下述任一条件时，即应首先发送预检请求：
- 使用了下面任一 HTTP 方法：
	- PUT
	- DELETE
	- CONNECT
	- OPTIONS
	- TRACE
	- PATCH
- 人为设置了对 CORS 安全的首部字段集合之外的其他首部字段。该集合为：
	- Accept
	- Accept-Language
	- Content-Language
	- Content-Type (but note the additional requirements below)
	- DPR
	- Downlink
	- Save-Data
	- Viewport-Width
	- Width
- Content-Type 的值不属于下列之一:
	- application/x-www-form-urlencoded
	- multipart/form-data
	- text/plain
- 请求中的XMLHttpRequestUpload 对象注册了任意多个事件监听器。
- 请求中使用了ReadableStream对象。

>注意: WebKit Nightly 和 Safari Technology Preview 为Accept, Accept-Language, 和 Content-Language 首部字段的值添加了额外的限制。如果这些首部字段的值是“非标准”的，WebKit/Safari 就不会将这些请求视为“简单请求”。WebKit/Safari 并没有在文档中列出哪些值是“非标准”的，不过我们可以在这里找到相关讨论：[Require preflight for non-standard CORS-safelisted request headers Accept, Accept-Language, and Content-Language][1], [Allow commas in Accept, Accept-Language, and Content-Language request headers for simple CORS][2], and [Switch to a blacklist model for restricted Accept headers in simple CORS requests][3]。其它浏览器并不支持这些额外的限制，因为它们不属于规范的一部分。

# 解决方法
- 修改浏览器设置 https://github.com/zhongxia245/blog/issues/28
- NGINX转发请求
- 修改AJAX请求content type为上面所允许的三个之一（未解决本文所涉及的问题）

# 参考文档

- [浏览器的同源策略][4]
- [HTTP访问控制（CORS）][5]

  [1]: https://bugs.webkit.org/show_bug.cgi?id=165178
  [2]: https://bugs.webkit.org/show_bug.cgi?id=165566
  [3]: https://bugs.webkit.org/show_bug.cgi?id=166363
  [4]: https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy 
  [5]: https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS
