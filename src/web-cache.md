transition:move

[slide]
# 浏览器缓存浅析
###### 作者：**徐德明**

[slide]
# 目录
* 缓存的介绍
	- 缓存的作用
	- 缓存的分类
* 浏览器缓存的特点
* 浏览器缓存策略图
* 浏览器缓存协议头介绍
	- Expires
	- Cache-Control
	- Last-Modified / If-Modified-Since
	- ETag / If-None-Match
* Etag出现的必要性
* 缓存的优先级、强制不缓存及更新缓存
* 浏览器缓存常规设置
* 浏览器缓存demo

[slide]
# 缓存的介绍

`缓存的作用：`
* 加快应用的打开速度，提升用户体验
* 节省带宽
* 减少对源服务器的压力

`缓存的分类：`
* **DNS缓存**：避免每次都解析域名；实际中我们会将CDN域名预解析，如：`<link rel="dns-prefetch" href="//xxx.com/">`
* **CDN内容分发缓存**：避免每次远距离、跨机房、跨运营商等获取静态资源甚至回源
* **Web服务器、反向代理缓存（如：Nginx、Apache、Tomcat等）**：避免每次访问源服务器
* **数据库缓存（如Redis）**：减少后端数据库的压力
* **终端缓存（如：浏览器缓存、C/S端缓存等）**：减少http请求
* etc...

[slide]
#浏览器缓存的特点
* 因为web应用都是B/S架构，大多基于http协议，所以浏览器缓存是和http协议相关的，不同的协议版本有区别
* 不同的浏览器、同样浏览器的不同版本对缓存的处理有点区别，比如有的浏览器是from cache，有的是304
* 缓存是保存在浏览器中的；所以同一个浏览器，不管新开tab或者重启都是from cache或者304；

[slide]
# 浏览器缓存策略图
![img](http://image.techweb.com.cn/upload/roll/2016/01/05/201601055112_3592.jpg)

[slide]
# 浏览器缓存协议头介绍
* **Expires**
	* 设置资源的过期时间

* **Cache-Control**
	* 设置资源的缓存时间

* **Last-Modified / If-Modified-Since**
	* 设置资源的最后修改时间

* **ETag / If-None-Match**
	* 设置资源的内容hash

* **etc...**
	* 比如Age头，应用不是很广泛，暂不讨论

[slide]
# Expires
* `简介：`
	- 其值为某个日期，规定这个日期之前都返回缓存，不请求服务器；
	- 无http请求
	- Expires = max-age + "每次下载时的当前的request时间"，所以一旦重新下载的页面后，expires就重新计算一次
* `缺点：`
	- 没过设置的时间时，URI没变化时，就算资源更新了，还是直接from cache
	- 其值为服务器端的时间，但是request的时间为终端时间，如果俩者时间相差很大(比如更改客户端时间)，那么误差就很大；所以Expires都要和Cache-Control配合使用
* `例子：`
	- Expires: Mon, 08 Jan 2018 03:24:01 GMT

[slide]
# Cache-Control
* `简介：`
	- **http/1.1**规范定义，其值为秒数，规定资源缓存多少秒，存储的是更新间隔；
	- 无http请求
	- 会和Expires配合使用以保证准确性；
* `缺点：`
	- 同样有Expires一样的缺点：没过设置的时间时，URI没变化时，就算资源更新了，还是直接from cache
* `例子：`
	- Cache-Control:max-age=15552000

[slide]
# Last-Modified
* `简介：`
	- 对应请求头中的**If-Modified-Since**;
	- 其值是某个日期，表示文档的最后修改时间;
	- 有http请求，若最后修改时间没变，直接返回304，只返回头部，不返回资源body，以节省带宽;
* `流程：`
	- 第一次请求时，服务器会将资源的最后修改时间返回给浏览器，这时浏览器会将其保存，再请求时，浏览器会发送**If-Modified-Since**到服务器进行对比，没变化返回304，否则返回200
* `缺点：`
	- 会影响性能，因为每次请求后端都需要获取文件夹的最后修改时间
* `例子：`
	- Last-Modified:Wed, 22 Nov 2017 03:54:37 GMT

[slide]
# ETag
* `简介：`
	- 对应请求头中的**If-None-Match**;
	- 其值是资源内容(全部内容或者部分内容)的hash值；
	- 有http请求，若资源内容没变，直接返回304，只返回头部，不返回资源body，以节省带宽;
* `流程：`
	- 第一次请求时，服务器会对资源内容进行hash，并把其传给浏览器，这时浏览器会将其保存，再请求时，浏览器会发送**If-None-Match**到服务器进行对比，没变化返回304，否则返回200
* `缺点：`
	- 会影响性能，因为每次请求后端都需要读取文件内容并hash
* `例子：`
	- Etag:QJ8ihycuJKpVTLT5xiQQhNT1Iec=

[slide]
# Etag出现的必要性
`Last-Modified已经足以让浏览器知道本地的缓存副本是否足够新，为什么还需要Etag？`

* **Last-Modified**标注的最后修改只能精确到秒级，如果资源在1秒内修改多次，它将不能准确标注资源的新鲜度
* 如果资源被改错后再回改成原来，这时资源内容并没有任何变化，但**Last-Modified**却改变了，导致文件没法使用缓存
* 有可能存在服务器没有准确获取文件修改时间，或者与代理服务器（如Nginx）时间不一致等情形
* 补充（受益于老板指导）：Etag更适用于动态生成又不太方便的静态化的资源


[slide]
# 缓存的优先级、强制不缓存及更新缓存

`优先级`
* Cache-Control > Expires > ETag / Last-Modified
* Last-Modified与ETag同时使用时，浏览器会同时发送If-Modified-Since和If-None-Match，按照http/1.1规范，服务端必须两个都验证通过后才能返回304

`强制不缓存`
* **Expires: 0** && **Cache-Control: 'no-cache,no-store'**
* 动态给资源添加版本控制，如：'index.js?v=' + Date.now()

`强制更新缓存`
* 如果文件变化了，我们可以给资源加版本号强制浏览器请求服务器(此时浏览器一定会下载资源返回200，因为URI变更后浏览器会当其为一个新资源)

[slide]
# 浏览器缓存常规设置
1. 首先设置`Expires` 和 `Cache-Control`，可以直接from cache (from memory cache 或 from disk cache），不用http request；
	- **from memory cache**直接从内存读取缓存，从chrome中看，其请求时间都为0；退出进程时（关闭浏览器），数据会被清空
	- **from disk cache**是从硬盘读取缓存，从chrome中看，其请求时间都大于0；退出进程时（关闭浏览器），数据不会被清空
2. 设置`Last-Modified`
	- Expires和Cache-Control只能控制资源缓存多久，但是过了缓存时间，资源还是有可能没变化，所以为了节省带宽，需要配合Last-Modified / ETag使用，这样没变化时就可以直接返回304了

`Tips:` **获取文件上次修改时间的消耗比拿到整个数据的消耗要小的多，所以很多时候只要Last-Modified就基本可以了，特殊情况采用Etag**

[slide]
# 浏览器缓存demo
`常规缓存设置 测试`
	1. 更改Mid框架中的配置文件，debug改为false
	2. 将静态文件服务器jserver.js中的maxAge改为20（秒）
	3. 启动Mid框架
	4. 观察network：
		- 第一次为200（第一次请求下载资源）
		- 第二为from cache（因为设置了Expires和Cache-Control）
		- 等待20秒后，刷新浏览器，此时静态资源都变成304了（此时Cache-Control超过了20秒，缓存过时了，浏览器不会from cache，而会请求服务器，但是资源没变化，所以命中了Last-Modified，返回304）

[slide]
# 浏览器缓存demo
`Etag 测试`
	1. node node.js
	2. 打开浏览器观察，第一次应该返回200，response headers中有Etag字段，request headers中没有If-None-Match字段
	3. 刷新页面，返回304，response headers中没有Etag字段，request headers中有If-None-Match字段，其值是第一次Etag的值
	4. 改变test.js的内容，再刷新页面，返回200，代表重新返回内容了，这时response headers中有Etag字段，但是值变了，request headers中有If-None-Match字段，其值还是第一次返回的Etag的值，两个值不一样
	5. 在刷新页面，结果和第三步一样

[slide]
# Q & A
















<style>
p{
	text-align: left;
}
img{
	display: block;
	width: 70%;
	margin: 0 auto;
}
ul ul {
	margin-top: 0;
}
ul li{
	line-height: 1.6em;
}
</style>
