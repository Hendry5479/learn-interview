# 11.浏览器基础知识点及常考面试题
## 事件机制
涉及面试题：事件的触发过程是怎么样的？知道什么是事件代理嘛？
事件是文档和浏览器窗口中发生的特定的交互瞬间
IE的事件流是冒泡流(event bubbling)：事开始时由具体元素接收，逐级向上传播到不具体的节点
Netscape是捕获流(event capturing)：不具体节点向下传播至具体元素

事件触发有三个阶段：
1.window 往事件触发处传播，遇到注册的捕获事件会触发
2.传播到事件触发处时触发注册的事件
3.从事件触发处往 window 传播，遇到注册的冒泡事件会触发

特例，如果给 body 的子节点同时注册冒泡和捕获事件，事件触发会按照注册的顺序执行

注册事件
通常使用 addEventListener 注册事件，第三个参数可以是布尔值或对象
对于布尔值 useCapture 参数，默认值为 false ，useCapture 决定事件是捕获还是冒泡
对于对象参数，可以使用以下几个属性
capture：布尔值，和 useCapture 作用一样
once：布尔值，值为 true 表示该回调只会调用一次，调用后移除监听
passive：布尔值，表示永远不会调用 preventDefault
如果希望事件只触发在目标上，可以使用 stopPropagation 阻止事件的进一步传播
stopPropagation 可以用来阻止事件冒泡和捕获
stopImmediatePropagation 同样也能阻止事件，还能阻止该事件目标执行别的注册事件

**事件代理：**
例如：li的事件注册在ul上
如果节点中的子节点是动态生成的，那么子节点的事件应该注册在父节点上
事件代理相较于直接注册的优点：节省内存、不需要注销事件


## 跨域
涉及面试题：什么是跨域？为什么浏览器要使用同源策略？有几种方式解决跨域问题？了解预检请求嘛


#### 同源策略
如果**协议、域名或者端口**有一个不同就是跨域，Ajax 请求会失败
因为浏览器出于安全考虑，有同源策略
具体的安全考虑： 防止 CSRF 攻击（Cross-site request forgery，利用用户的登录态发起恶意请求）
没有同源策略，A 网站可以被任意其他来源的 Ajax 访问到内容
如果 A 网站存在登录态，对方可以通过 Ajax 获得你的任何信息
当然跨域并不能完全阻止 CSRF
跨域的请求其实已经发出去了
请求必然是发出去了，但是浏览器拦截了响应。
通过表单的方式可以发起跨域请求，为什么 Ajax 就不会?
因为跨域是为了阻止用户读取到另一个域名下的内容，Ajax 可以获取响应，浏览器认为这不安全，所以拦截了响应。但是表单并不会获取新的内容，所以可以发起跨域请求。
同时也说明了跨域并不能完全阻止 CSRF，因为请求毕竟是发出去了

#### 解决跨域问题的方式
#### JSONP
利用` script `标签没有跨域限制的漏洞，指向需要访问的地址并提供回调函数来接收数据
JSONP 使用简单且兼容性不错，但是只限于 get 请求

``` JavaScript
function jsonp(url, jsonpCallback, success) {   
  let script = document.createElement('script')   
  script.src = url   
  script.async = true   
  script.type = 'text/javascript'   
  window[jsonpCallback] = function(data) {     
    success && success(data)   
  }   
  document.body.appendChild(script) } 
  jsonp('http://xxx', 'callback', function(value) {   
    console.log(value) 
})
```
#### CORS
服务端设置 Access-Control-Allow-Origin 开启 CORS
该属性表示哪些域名可以访问资源，如果设置通配符则表示所有网站都可以访问资源。
通过这种方式解决跨域问题的话，会在发送请求时出现两种情况，分别为简单请求和复杂请求

#### 简单请求
以 Ajax 为例，当满足以下条件时，会触发简单请求
13.	使用下列方法之一：
	GET
	HEAD
	POST
14.	Content-Type 的值仅限于下列三者之一：
	text/plain
	multipart/form-data
	application/x-www-form-urlencoded
请求中的任意 XMLHttpRequestUpload 对象均没有注册任何事件监听器
XMLHttpRequestUpload 对象可以使用 XMLHttpRequest.upload 属性访问

#### 复杂请求
首先发起预检请求，该请求是 option 方法，通过该请求知道服务端是否允许跨域请求
对于预检请求，Node 设置 CORS会遇到一个坑
以 express 框架举例：
app.use((req, res, next) => {   res.header('Access-Control-Allow-Origin', '*')   res.header('Access-Control-Allow-Methods', 'PUT, GET, POST, DELETE, OPTIONS')   res.header(     'Access-Control-Allow-Headers',     'Origin, X-Requested-With, Content-Type, Accept, Authorization, Access-Control-Allow-Credentials')   next() })

该请求会验证你的 Authorization 字段，没有的话就会报错。
当前端发起了复杂请求后，你会发现就算你代码是正确的，返回结果也永远是报错的。因为预检请求也会进入回调中，也会触发 next 方法，因为预检请求并不包含 Authorization 字段，所以服务端会报错。
想解决这个问题很简单，只需要在回调中过滤 option 方法即可
res.statusCode = 204res.setHeader('Content-Length', '0') res.end()

document.domain
该方式只能用于二级域名相同的情况下，比如 a.test.com 和 b.test.com 适用于该方式。
只需要给页面添加 document.domain = 'test.com' 表示二级域名都相同就可以实现跨域

postMessage
这种方式通常用于获取嵌入页面中的第三方页面数据。一个页面发送消息，另一个页面判断来源并接收消息
// 发送消息端window.parent.postMessage('message', 'http://test.com')// 接收消息端var mc = new MessageChannel() mc.addEventListener('message', event => {   var origin = event.origin || event.originalEvent.origin   if (origin === 'http://test.com') {     console.log('验证通过')   } })

### 存储
涉及面试题：有几种方式实现存储功能，分别有什么优缺点？什么是 Service Worker？

#### cookie，localStorage，sessionStorage，indexDB
cookie 已经不建议用于存储
如果没有大量存储需求，可以使用 localStorage 和 sessionStorage 
对于不怎么改变的数据尽量使用 localStorage ，否则用 sessionStorage 
cookie安全性
value：保存登录态的话要加密
http-only：不能通过js访问，减少xss(Cross Site Scripting) 跨站脚本攻击
xss：使用js程序盗用cookie
secure：智能在HTTPS协议的请求中携带
same-site：规定浏览器不能在跨域请求中携带Cookie，减少CSRF攻击
#### Service Worker
Service Worker 是运行在浏览器背后的独立线程，一般用来实现缓存功能
传输协议必须为 HTTPS，因为 Service Worker 涉及请求拦截，所以保障安全
Service Worker 实现缓存的三个步骤：
1.注册 Service Worker
2.监听到 install 事件以后缓存文件
3.下次访问的时候通过拦截请求的方式查询是否存在缓存，存在则读取缓存，否则请求数据

源码待续。。。


### 12.浏览器缓存机制
注意：该知识点属于性能优化领域，并且整一章节都是一个面试题。
缓存是性能优化中简单高效的一种方式，可以显著减少网络传输所带来的损耗
对于数据请求来说，可以分为发起网络请求、后端处理、浏览器响应三个步骤。
浏览器缓存帮助我们在第一和第三步骤中优化性能。
比如说直接使用缓存而不发起请求，或者发起了请求但后端存储的数据和前端一致，那么就没有必要再将数据回传回来，这样就减少了响应数据。

#### 缓存位置
从缓存位置来说分为四种，并且有优先级，当依次查找缓存且都没有命中的时候才请求网络
Service Worker
Memory Cache
Disk Cache
Push Cache
网络请求

#### Service Worker
Service Worker 的缓存与其他缓存机制不同，它可以自由控制缓存哪些文件、如何匹配缓存、如何读取缓存，并且缓存是持续性的。
Service Worker 没有命中缓存的时候，调用 fetch 函数获取数据
如果没有在 Service Worker 命中缓存的话，会根据缓存查找优先级去查找数据
但是不管从 Memory Cache 还是从网络请求中获取数据，浏览器都显示从 Service Worker 中获取的内容


#### Memory Cache
读取内存中的数据比磁盘快，可是持续性短，会随着进程的释放而释放。 一旦关闭 Tab 页面，内存的缓存也就被释放了。

#### Disk Cache
读取速度慢点，但是比之 Memory Cache 胜在容量和存储时效性上
在所有浏览器缓存中，Disk Cache 覆盖面最大
它会根据 HTTP Header 的字段判断哪些资源需要缓存，哪些资源可以不请求直接使用，哪些资源已经过期需要重新请求。并且即使在跨站点的情况下，相同地址的资源一旦被硬盘缓存下来，就不会再次去请求数据

#### Push Cache
Push Cache 是 HTTP/2 中的内容，当以上三种缓存都没有命中时，它才会被使用。并且缓存时间也很短暂，只在会话（Session）中存在，一旦会话结束就被释放。

所有的资源都能被推送，但是 Edge 和 Safari 浏览器兼容性不好
可以推送 no-cache 和 no-store 的资源
一旦连接被关闭，Push Cache 就被释放
多个页面可以使用相同的 HTTP/2 连接，也就是使用同样的缓存
Push Cache 的缓存只能被使用一次
浏览器可以拒绝接受已经存在的资源推送
你可以给其他域名推送资源

#### 缓存策略
通常缓存策略分为两种：强缓存和协商缓存，缓存策略通过设置 HTTP Header 实现

#### 强缓存
强缓存通过设置两种 HTTP Header 实现：Expires 和 Cache-Control 
强缓存表示在缓存期间不需要请求，state code 为 200
Expires
Expires: Wed, 22 Oct 2018 08:41:00 GMT
Expires 是 HTTP/1 的产物，表示资源会在 22 Oct 2018 08:41:00 后过期，需要再次请求。
Expires 受限于本地时间，如果修改了本地时间，可能会造成缓存失效

Cache-control
Cache-control: max-age=30
Cache-Control 出现于 HTTP/1.1，优先级高于 Expires 
该属性值表示资源会在 30 秒后过期，需要再次请求
Cache-Control 可以在请求头或者响应头中设置，并且可以组合使用多种指令

#### 协商缓存
如果缓存过期了，就需要发起请求验证资源是否更新
协商缓存通过设置两种 HTTP Header 实现：Last-Modified 和 ETag 
当浏览器请求验证资源时，如果资源没改变，服务端返回 304 状态码，并更新缓存有效期

Last-Modified 和 If-Modified-Since
Last-Modified 表示本地文件最后修改日期，If-Modified-Since 会将 Last-Modified 的值发送给服务器，询问服务器在该日期后资源是否有更新，有更新的话就会将新的资源发送回来，否则返回 304 状态码。
Last-Modified 的弊端：
如果本地打开缓存文件，即使没有修改， Last-Modified还是会被修改，服务端不能命中缓存导致发送相同的资源
 Last-Modified 以秒计时，如果在不可感知的时间内修改文件，服务端会认为资源还是命中了，不会返回正确的资源

因为以上弊端，在 HTTP / 1.1 出现了 ETag 
ETag 和 If-None-Match
ETag 类似于文件指纹，If-None-Match 会将当前 ETag 发送给服务器，询问该资源 ETag 是否变动，有变动的话就将新的资源发送回来。并且 ETag 优先级比 Last-Modified 高。

如果缓存策略没设置，浏览器会怎么处理？
浏览器会采用一个启发式的算法，（响应头的 Date  -  Last-Modified 的 10% ）作为缓存时间

#### 实际场景应用缓存策略
频繁变动的资源
首先使用 Cache-Control: no-cache 使浏览器每次都请求服务器，然后配合 ETag 或者 Last-Modified 来验证资源是否有效。这样的做法虽然不能节省请求数量，但是能显著减少响应数据大小。

代码文件
HTML 文件一般不缓存或者缓存时间很短
现在使用工具来打包代码，可以对文件名进行哈希处理，代码修改后才会生成新的文件名
给代码文件设置缓存有效期一年 Cache-Control: max-age=31536000
只有当 HTML 文件引入的文件名发生改变才会下载最新的代码文件，否则使用缓存


# 13.浏览器渲染原理
注意：该章节都是一个面试题。
渲染引擎：Firefox Gecko， Chrome & Safari WebKit

浏览器接收到 HTML 文件并转换为 DOM 树

字节数据 => 字符串 =(词法分析，标记化)=> 标记(Token) => Node => DOM Tree

将 CSS 文件转换为 CSSOM 树
字节数据 => 字符串 =(词法分析，标记化)=> 标记(Token) => Node => 	CSSOM  

### 为什么操作 DOM 慢
因为 DOM 是属于渲染引擎的东西，而 JS 是 JS 引擎的东西。
JS 操作 DOM 涉及两个线程间的通信，势必带来性能损耗。
操作 DOM 次数多就等同于一直在进行线程间的通信，并且操作 DOM 会带来重绘回流
所以导致性能问题

**经典面试题：插入几万个 DOM，如何实现页面不卡顿？**
首先肯定不能一次性全部插入，这样肯定会卡顿，所以重点应该是如何分批次部分渲染 DOM
1.requestAnimationFrame循环的插入 DOM
2.虚拟滚动（virtualized scroller）
原理：只渲染可视区域内容，非可见区域完全不渲染，用户滚动实时替换渲染的内容

什么情况阻塞渲染
渲染的前提是生成渲染树，所以 HTML 和 CSS 肯定会阻塞渲染
1.如果想渲染快，应该降低一开始需要渲染的文件大小，并且扁平层级，优化选择器
2.当浏览器解析 script 标签时，会暂停构建 DOM。如果想首屏渲染得快，就不应该在首屏加载 JS 文件，这也是建议将 script 放在 body 标签底部的原因
除了放在底部，还可以给 script 标签添加 defer 或者 async 属性
defer 属性表示JS 文件会并行下载，但是会在 HTML 解析完成后顺序执行
async 属性表示 JS 文件下载和解析不会阻塞渲染

### 重绘（Repaint）和回流（Reflow）
重绘和回流会在设置节点样式时频繁出现，同时影响性能
	重绘：节点需要更改外观而不影响布局，比如改变 color 
	回流：布局或者几何属性需要改变
回流必定重绘，重绘不一定回流
回流所需的成本比重绘高的多，改变子节点可能导致父节点回流
以下动作可能会导致性能问题：
改变 window 大小
改变字体
添加或删除样式
文字改变
定位或者浮动
盒模型
很多人不知道，重绘和回流和 Eventloop 有关
Eventloop 执行完 Microtasks 后，会判断 document 是否更新，浏览器是 60Hz 的刷新率，每 16.6ms 才会更新一次
然后判断是否有 resize 或者 scroll 事件，有的话触发事件，所以  resize 和 scroll 事件也是至少 16ms 才触发一次，并自带节流功能
判断是否触发了 media query
更新动画并且发送事件
判断是否有全屏操作事件
执行 requestAnimationFrame 回调
执行 IntersectionObserver 回调，该方法用于判断元素是否可见，可以用于懒加载
更新界面
以上是一帧中可能会做的事情。如果在一帧中有空闲时间，就会执行 requestIdleCallback 

既然我们已经知道了重绘和回流会影响性能，那么接下来我们将会来学习如何减少重绘和回流的次数。

### 减少重绘和回流
使用 transform 替代 top
``` JavaScript
<div class="test"></div><style>   
.test {     
  position: absolute;     
  top: 10px;     
  width: 100px;     
  height: 100px;     
  background: red;   
}
</style>

<script>   
setTimeout(() => {     // 引起回流     
  document.querySelector('.test').style.top = '100px'
  }, 1000)
</script>
```
使用 visibility 替换 display: none ，因为前者只引起重绘，后者引发回流（改变了布局）
不要把节点属性值放在循环里当成循环里的变量
``` JavaScript
for(let i = 0; i < 1000; i++) {     
  // 获取 offsetTop 会导致回流，因为需要去获取正确的值     
  console.log(document.querySelector('.test').style.offsetTop) 
}
```
不要使用 table 布局，可能造成整个 table 的重新布局
动画实现的速度选择，速度越快，回流越多，可以选择使用 requestAnimationFrame
CSS 选择符从右往左匹配查找，避免节点层级过多
将频繁重绘或者回流的节点设置为图层，图层能阻止该节点的渲染行为影响别的节点。比如对于 video 标签，浏览器会自动将该节点变为图层
设置节点为图层的方式可以通过以下几个属性，生成新图层
will-change
video、iframe 标签

思考题：在不考虑缓存和优化网络协议的前提下，考虑可以通过哪些方式来最快渲染页面，也就是常说的关键渲染路径，这部分也是性能优化中的一块内容。

怎么测量有没有加快渲染速度呢
 
发生 DOMContentLoaded 事件后会生成渲染树，生成渲染树就可以进行渲染了，这一过程更大程度上和硬件有关系了

提示如何加速：
从文件大小考虑
从 script 标签使用上来考虑
从 CSS、HTML 的代码书写上来考虑
从需要下载的内容是否需要在首屏使用上来考虑


# 14.安全防范知识点
### XSS
涉及面试题：什么是 XSS 攻击？如何防范 XSS 攻击？什么是 CSP？
XSS：Cross Site Scripting 攻击者将代码注入网页中执行
XSS 分为两类：持久型和非持久型
持久型：代码被服务端写入数据库，这种攻击危害性很大，导致大量用户都受到攻击
举个例子，对于评论功能就得防范持久型 XSS 
非持久型：通过修改 URL 参数加入攻击代码，诱导用户访问链接从而攻击

**转义字符**
首先，对于用户的输入应该是永远不信任的。
最普遍的做法就是转义输入输出的内容，对于引号、尖括号、斜杠进行转义


**白名单过滤**
也可以通过黑名单过滤，但考虑到需要过滤的标签和标签属性太多，更推荐白名单
使用 js-xss实现

**CSP**
CSP 本质是建立白名单，开发者告诉浏览器哪些外部资源可以执行
只需要配置规则，如何拦截是由浏览器实现，从而减少 XSS 攻击
开启 CSP：
33.	设置 HTTP Header 的 Content-Security-Policy
34.	设置 meta 标签的方式 <meta http-equiv="Content-Security-Policy">

### Cross-site request forgery 跨站请求伪造
涉及面试题：什么是 CSRF 攻击？如何防范 CSRF 攻击？


原理：攻击者构造后端请求地址，诱导用户点击发起请求。如果用户处于登录态，后端会进行相应的逻辑
举个例子，网站有一个通过 GET 请求提交用户评论的接口，攻击者可以在钓鱼网站中加入图片，图片地址就是评论接口
如何防御
防范 CSRF 的几种规则：
Get 请求不对数据修改
不让第三方网站访问到用户 Cookie
阻止第三方网站请求接口
请求时附带验证信息，比如验证码或者 Token

SameSite
对 Cookie 设置 SameSite 属性，表示 Cookie 不随着跨域请求发送

验证 Referer
通过验证 Referer 判断该请求是否为第三方网站发起
Token
服务器下发一个随机 Token，每次发起请求将 Token 携带上，服务器验证 Token

点击劫持
涉及面试题：什么是点击劫持？如何防范点击劫持？
点击劫持是一种视觉欺骗的攻击手段
攻击者将网站通过 iframe 嵌套的方式嵌入自己的网页，并将 iframe 设置为透明，在页面中透出按钮诱导用户点击

防御的方法：
X-FRAME-OPTIONS
X-FRAME-OPTIONS 是一个 HTTP 响应头，防御用 iframe 嵌套的点击劫持攻击
该响应头有三个值可选：
	DENY，表示页面不允许通过 iframe 展示
	SAMEORIGIN，表示页面可以在相同域名下通过 iframe 展示
	ALLOW-FROM，表示页面可以在指定来源的 iframe 中展示
JS 防御
对于远古浏览器来说，只有通过 JS 的方式来防御点击劫持

### 中间人攻击
涉及面试题：什么是中间人攻击？如何防范中间人攻击？
**定义**：攻击方同时与服务端和客户端建立连接，获得双方的通信信息，还能修改通信信息
不建议使用公共 Wi-Fi，因为很可能就会发生中间人攻击
**防御中间人攻击**：增加安全通道来传输信息，例如HTTPS



# 15.从 V8 中看 JS 性能优化
注意：该知识点属于性能优化领域。

性能问题越来越成为前端火热的话题，因为随着项目的逐步变大，性能问题也逐步体现出来。为了提高用户的体验，减少加载时间，工程师们想尽一切办法去优化细节。

掘金之前已经出过一本关于性能的小册，我在写涉及性能优化的内容之前就特地去购买了这本小册阅读，目的是为了写出点不一样的东西。当然性能优化归结起来还是那几个点，我只能尽可能地写出那本小册没有提及的内容，部分内容还是会有重叠的。当然它通过了十五个章节去介绍性能，肯定会讲的比我细，有兴趣的可以同时购买还有本 「前端性能优化原理与实践 」小册，形成一个互补。

在这几个章节中不会提及浏览器、Webpack、网络协议这几块如何优化的内容，因为对应的模块中已经讲到了这部分的内容，如果你想学习这几块该如何性能优化的话，可以去对应的章节阅读。

在这一章节中我们将来学习如何让 V8 优化我们的代码，下一章节将会学习性能优化剩余的琐碎点，因为性能优化这个领域所涉及的内容都很碎片化。

在学习如何性能优化之前，我们先来了解下如何测试性能问题，毕竟是先有问题才会去想着该如何改进。

### 测试性能工具
Chrome 已经提供了一个大而全的性能测试工具 Audits


Audits 所处位置
点我们点击 Audits 后，可以看到如下的界面


Audits 界面
在这个界面中，我们可以选择想测试的功能然后点击 Run audits ，工具就会自动运行帮助我们测试问题并且给出一个完整的报告


Audits 工具给出的报告
上图是给掘金首页测试性能后给出的一个报告，可以看到报告中分别为性能、体验、SEO 都给出了打分，并且每一个指标都有详细的评估


指标中的详细评估
评估结束后，工具还提供了一些建议便于我们提高这个指标的分数


优化建议
我们只需要一条条根据建议去优化性能即可。

除了 Audits 工具之外，还有一个 Performance 工具也可以供我们使用。


Performance 工具给出的报告
在这张图中，我们可以详细的看到每个时间段中浏览器在处理什么事情，哪个过程最消耗时间，便于我们更加详细的了解性能瓶颈。

### JS 性能优化
JS 是编译型还是解释型语言其实并不固定。首先 JS 需要有引擎才能运行起来，无论是浏览器还是在 Node 中，这是解释型语言的特性。但是在 V8 引擎下，又引入了 TurboFan 编译器，他会在特定的情况下进行优化，将代码编译成执行效率更高的 Machine Code，当然这个编译器并不是 JS 必须需要的，只是为了提高代码执行性能，所以总的来说 JS 更偏向于解释型语言。

那么这一小节的内容主要会针对于 Chrome 的 V8 引擎来讲解。

在这一过程中，JS 代码首先会解析为抽象语法树（AST），然后会通过解释器或者编译器转化为 Bytecode 或者 Machine Code


V8 转化代码的过程
从上图中我们可以发现，JS 会首先被解析为 AST，解析的过程其实是略慢的。代码越多，解析的过程也就耗费越长，这也是我们需要压缩代码的原因之一。另外一种减少解析时间的方式是预解析，会作用于未执行的函数，这个我们下面再谈。


2016 年手机解析 JS 代码的速度
这里需要注意一点，对于函数来说，应该尽可能避免声明嵌套函数（类也是函数），因为这样会造成函数的重复解析。

function test1() {
  // 会被重复解析
  function test2() {}
}
然后 Ignition 负责将 AST 转化为 Bytecode，TurboFan 负责编译出优化后的 Machine Code，并且 Machine Code 在执行效率上优于 Bytecode


那么我们就产生了一个疑问，什么情况下代码会编译为 Machine Code？

JS 是一门动态类型的语言，并且还有一大堆的规则。简单的加法运算代码，内部就需要考虑好几种规则，比如数字相加、字符串相加、对象和字符串相加等等。这样的情况也就势必导致了内部要增加很多判断逻辑，降低运行效率。

function test(x) {
  return x + x
}

test(1)
test(2)
test(3)
test(4)
对于以上代码来说，如果一个函数被多次调用并且参数一直传入 number 类型，那么 V8 就会认为该段代码可以编译为 Machine Code，因为你固定了类型，不需要再执行很多判断逻辑了。

但是如果一旦我们传入的参数类型改变，那么 Machine Code 就会被 DeOptimized 为 Bytecode，这样就有性能上的一个损耗了。所以如果我们希望代码能多的编译为 Machine Code 并且 DeOptimized 的次数减少，就应该尽可能保证传入的类型一致。

那么你可能会有一个疑问，到底优化前后有多少的提升呢，接下来我们就来实践测试一下到底有多少的提升。

const { performance, PerformanceObserver } = require('perf_hooks')

function test(x) {
  return x + x
}
// node 10 中才有 PerformanceObserver
// 在这之前的 node 版本可以直接使用 performance 中的 API
const obs = new PerformanceObserver((list, observer) => {
  console.log(list.getEntries())
  observer.disconnect()
})
obs.observe({ entryTypes: ['measure'], buffered: true })

performance.mark('start')

let number = 10000000
// 不优化代码
%NeverOptimizeFunction(test)

while (number--) {
  test(1)
}

performance.mark('end')
performance.measure('test', 'start', 'end')
以上代码中我们使用了 performance API，这个 API 在性能测试上十分好用。不仅可以用来测量代码的执行时间，还能用来测量各种网络连接中的时间消耗等等，并且这个 API 也可以在浏览器中使用。


优化与不优化代码之间的巨大差距
从上图中我们可以发现，优化过的代码执行时间只需要 9ms，但是不优化过的代码执行时间却是前者的二十倍，已经接近 200ms 了。在这个案例中，我相信大家已经看到了 V8 的性能优化到底有多强，只需要我们符合一定的规则书写代码，引擎底层就能帮助我们自动优化代码。

另外，编译器还有个骚操作 Lazy-Compile，当函数没有被执行的时候，会对函数进行一次预解析，直到代码被执行以后才会被解析编译。对于上述代码来说，test 函数需要被预解析一次，然后在调用的时候再被解析编译。但是对于这种函数马上就被调用的情况来说，预解析这个过程其实是多余的，那么有什么办法能够让代码不被预解析呢？

其实很简单，我们只需要给函数套上括号就可以了

(function test(obj) {
  return x + x
})
但是不可能我们为了性能优化，给所有的函数都去套上括号，并且也不是所有函数都需要这样做。我们可以通过 optimize-js 实现这个功能，这个库会分析一些函数的使用情况，然后给需要的函数添加括号，当然这个库很久没人维护了，如果需要使用的话，还是需要测试过相关内容的。

### 小结
总结一下这一章节我们学习的知识

可以通过 Audit 工具获得网站的多个指标的性能报告
可以通过 Performance 工具了解网站的性能瓶颈
可以通过 Performance API 具体测量时间
为了减少编译时间，我们可以采用减少代码文件的大小或者减少书写嵌套函数的方式
为了让 V8 优化代码，我们应该尽可能保证传入参数的类型一致。这也给我们带来了一个思考，这是不是也是使用 TypeScript 能够带来的好处之一

# 16.性能优化琐碎事
注意：该知识点属于性能优化领域。

总的来说性能优化这个领域的很多内容都很碎片化，这一章节我们将来学习这些碎片化的内容。

### 图片优化
计算图片大小
对于一张 100 * 100 像素的图片来说，图像上有 10000 个像素点，如果每个像素的值是 RGBA 存储的话，那么也就是说每个像素有 4 个通道，每个通道 1 个字节（8 位 = 1个字节），所以该图片大小大概为 39KB（10000 * 1 * 4 / 1024）。

但是在实际项目中，一张图片可能并不需要使用那么多颜色去显示，我们可以通过减少每个像素的调色板来相应缩小图片的大小。

了解了如何计算图片大小的知识，那么对于如何优化图片，想必大家已经有 2 个思路了：

减少像素点
减少每个像素点能够显示的颜色
图片加载优化
不用图片。很多时候会使用到很多修饰类图片，其实这类修饰图片完全可以用 CSS 去代替。
对于移动端来说，屏幕宽度就那么点，完全没有必要去加载原图浪费带宽。一般图片都用 CDN 加载，可以计算出适配屏幕的宽度，然后去请求相应裁剪好的图片。
小图使用 base64 格式
将多个图标文件整合到一张图片中（雪碧图）
选择正确的图片格式：
对于能够显示 WebP 格式的浏览器尽量使用 WebP 格式。因为 WebP 格式具有更好的图像数据压缩算法，能带来更小的图片体积，而且拥有肉眼识别无差异的图像质量，缺点就是兼容性并不好
小图使用 PNG，其实对于大部分图标这类图片，完全可以使用 SVG 代替
照片使用 JPEG
DNS 预解析
DNS 解析也是需要时间的，可以通过预解析的方式来预先获得域名所对应的 IP。

<link rel="dns-prefetch" href="//yuchengkai.cn">
### 节流
考虑一个场景，滚动事件中会发起网络请求，但是我们并不希望用户在滚动过程中一直发起请求，而是隔一段时间发起一次，对于这种情况我们就可以使用节流。

理解了节流的用途，我们就来实现下这个函数

// func是用户传入需要防抖的函数
// wait是等待时间
const throttle = (func, wait = 50) => {
  // 上一次执行该函数的时间
  let lastTime = 0
  return function(...args) {
    // 当前时间
    let now = +new Date()
    // 将当前时间和上一次执行函数时间对比
    // 如果差值大于设置的等待时间就执行函数
    if (now - lastTime > wait) {
      lastTime = now
      func.apply(this, args)
    }
  }
}

setInterval(
  throttle(() => {
    console.log(1)
  }, 500),
  1
)
### 防抖
考虑一个场景，有一个按钮点击会触发网络请求，但是我们并不希望每次点击都发起网络请求，而是当用户点击按钮一段时间后没有再次点击的情况才去发起网络请求，对于这种情况我们就可以使用防抖。

理解了防抖的用途，我们就来实现下这个函数

// func是用户传入需要防抖的函数
// wait是等待时间
const debounce = (func, wait = 50) => {
  // 缓存一个定时器id
  let timer = 0
  // 这里返回的函数是每次用户实际调用的防抖函数
  // 如果已经设定过定时器了就清空上一次的定时器
  // 开始一个新的定时器，延迟执行用户传入的方法
  return function(...args) {
    if (timer) clearTimeout(timer)
    timer = setTimeout(() => {
      func.apply(this, args)
    }, wait)
  }
}
### 预加载
在开发中，可能会遇到这样的情况。有些资源不需要马上用到，但是希望尽早获取，这时候就可以使用预加载。

预加载其实是声明式的 fetch ，强制浏览器请求资源，并且不会阻塞 onload 事件，可以使用以下代码开启预加载

<link rel="preload" href="http://example.com">
预加载可以一定程度上降低首屏的加载时间，因为可以将一些不影响首屏但重要的文件延后加载，唯一缺点就是兼容性不好。

### 预渲染
可以通过预渲染将下载的文件预先在后台渲染，可以使用以下代码开启预渲染

<link rel="prerender" href="http://example.com"> 
预渲染虽然可以提高页面的加载速度，但是要确保该页面大概率会被用户在之后打开，否则就是白白浪费资源去渲染。

### 懒执行
懒执行就是将某些逻辑延迟到使用时再计算。该技术可以用于首屏优化，对于某些耗时逻辑并不需要在首屏就使用的，就可以使用懒执行。懒执行需要唤醒，一般可以通过定时器或者事件的调用来唤醒。

### 懒加载
懒加载就是将不关键的资源延后加载。

懒加载的原理就是只加载自定义区域（通常是可视区域，但也可以是即将进入可视区域）内需要加载的东西。对于图片来说，先设置图片标签的 src 属性为一张占位图，将真实的图片资源放入一个自定义属性中，当进入自定义区域时，就将自定义属性替换为 src 属性，这样图片就会去下载资源，实现了图片懒加载。

懒加载不仅可以用于图片，也可以使用在别的资源上。比如进入可视区域才开始播放视频等等。

### CDN
CDN 的原理是尽可能的在各个地方分布机房缓存数据，这样即使我们的根服务器远在国外，在国内的用户也可以通过国内的机房迅速加载资源。

因此，我们可以将静态资源尽量使用 CDN 加载，由于浏览器对于单个域名有并发请求上限，可以考虑使用多个 CDN 域名。并且对于 CDN 加载静态资源需要注意 CDN 域名要与主站不同，否则每次请求都会带上主站的 Cookie，平白消耗流量。

小结
这些碎片化的性能优化点看似很短，但是却能在出现性能问题时简单高效的提高性能，并且好几个点都是面试高频考点，比如节流、防抖。如果你还没有在项目中使用过这些技术，可以尝试着用到项目中，体验下功效。


# 17.Webpack 性能优化
在这一的章节中，我不会浪费篇幅给大家讲如何写配置文件。如果你想学习这方面的内容，那么完全可以去官网学习。在这部分的内容中，我们会聚焦于以下两个知识点，并且每一个知识点都属于高频考点：

有哪些方式可以减少 Webpack 的打包时间
有哪些方式可以让 Webpack 打出来的包更小
### 减少 Webpack 打包时间
优化 Loader
对于 Loader 来说，影响打包效率首当其冲必属 Babel 了。因为 Babel 会将代码转为字符串生成 AST，然后对 AST 继续进行转变最后再生成新的代码，项目越大，转换代码越多，效率就越低。当然了，我们是有办法优化的。

首先我们可以优化 Loader 的文件搜索范围

module.exports = {
  module: {
    rules: [
      {
        // js 文件才使用 babel
        test: /\.js$/,
        loader: 'babel-loader',
        // 只在 src 文件夹下查找
        include: [resolve('src')],
        // 不会去查找的路径
        exclude: /node_modules/
      }
    ]
  }
}
对于 Babel 来说，我们肯定是希望只作用在 JS 代码上的，然后 node_modules 中使用的代码都是编译过的，所以我们也完全没有必要再去处理一遍。

当然这样做还不够，我们还可以将 Babel 编译过的文件缓存起来，下次只需要编译更改过的代码文件即可，这样可以大幅度加快打包时间

loader: 'babel-loader?cacheDirectory=true'
HappyPack
受限于 Node 是单线程运行的，所以 Webpack 在打包的过程中也是单线程的，特别是在执行 Loader 的时候，长时间编译的任务很多，这样就会导致等待的情况。

HappyPack 可以将 Loader 的同步执行转换为并行的，这样就能充分利用系统资源来加快打包效率了

module: {
  loaders: [
    {
      test: /\.js$/,
      include: [resolve('src')],
      exclude: /node_modules/,
      // id 后面的内容对应下面
      loader: 'happypack/loader?id=happybabel'
    }
  ]
},
plugins: [
  new HappyPack({
    id: 'happybabel',
    loaders: ['babel-loader?cacheDirectory'],
    // 开启 4 个线程
    threads: 4
  })
]
DllPlugin
DllPlugin 可以将特定的类库提前打包然后引入。这种方式可以极大的减少打包类库的次数，只有当类库更新版本才有需要重新打包，并且也实现了将公共代码抽离成单独文件的优化方案。

接下来我们就来学习如何使用 DllPlugin

// 单独配置在一个文件中
// webpack.dll.conf.js
const path = require('path')
const webpack = require('webpack')
module.exports = {
  entry: {
    // 想统一打包的类库
    vendor: ['react']
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].dll.js',
    library: '[name]-[hash]'
  },
  plugins: [
    new webpack.DllPlugin({
      // name 必须和 output.library 一致
      name: '[name]-[hash]',
      // 该属性需要与 DllReferencePlugin 中一致
      context: __dirname,
      path: path.join(__dirname, 'dist', '[name]-manifest.json')
    })
  ]
}
然后我们需要执行这个配置文件生成依赖文件，接下来我们需要使用 DllReferencePlugin 将依赖文件引入项目中

// webpack.conf.js
module.exports = {
  // ...省略其他配置
  plugins: [
    new webpack.DllReferencePlugin({
      context: __dirname,
      // manifest 就是之前打包出来的 json 文件
      manifest: require('./dist/vendor-manifest.json'),
    })
  ]
}
代码压缩
在 Webpack3 中，我们一般使用 UglifyJS 来压缩代码，但是这个是单线程运行的，为了加快效率，我们可以使用 webpack-parallel-uglify-plugin 来并行运行 UglifyJS，从而提高效率。

在 Webpack4 中，我们就不需要以上这些操作了，只需要将 mode 设置为 production 就可以默认开启以上功能。代码压缩也是我们必做的性能优化方案，当然我们不止可以压缩 JS 代码，还可以压缩 HTML、CSS 代码，并且在压缩 JS 代码的过程中，我们还可以通过配置实现比如删除 console.log 这类代码的功能。

一些小的优化点
我们还可以通过一些小的优化点来加快打包速度

resolve.extensions：用来表明文件后缀列表，默认查找顺序是 ['.js', '.json']，如果你的导入文件没有添加后缀就会按照这个顺序查找文件。我们应该尽可能减少后缀列表长度，然后将出现频率高的后缀排在前面
resolve.alias：可以通过别名的方式来映射一个路径，能让 Webpack 更快找到路径
module.noParse：如果你确定一个文件下没有其他依赖，就可以使用该属性让 Webpack 不扫描该文件，这种方式对于大型的类库很有帮助
减少 Webpack 打包后的文件体积
注意：该内容也属于性能优化领域。

按需加载
想必大家在开发 SPA 项目的时候，项目中都会存在十几甚至更多的路由页面。如果我们将这些页面全部打包进一个 JS 文件的话，虽然将多个请求合并了，但是同样也加载了很多并不需要的代码，耗费了更长的时间。那么为了首页能更快地呈现给用户，我们肯定是希望首页能加载的文件体积越小越好，这时候我们就可以使用按需加载，将每个路由页面单独打包为一个文件。当然不仅仅路由可以按需加载，对于 loadash 这种大型类库同样可以使用这个功能。

按需加载的代码实现这里就不详细展开了，因为鉴于用的框架不同，实现起来都是不一样的。当然了，虽然他们的用法可能不同，但是底层的机制都是一样的。都是当使用的时候再去下载对应文件，返回一个 Promise，当 Promise 成功以后去执行回调。

Scope Hoisting
Scope Hoisting 会分析出模块之间的依赖关系，尽可能的把打包出来的模块合并到一个函数中去。

比如我们希望打包两个文件

// test.js
export const a = 1
// index.js
import { a } from './test.js'
对于这种情况，我们打包出来的代码会类似这样

[
  /* 0 */
  function (module, exports, require) {
    //...
  },
  /* 1 */
  function (module, exports, require) {
    //...
  }
]
但是如果我们使用 Scope Hoisting 的话，代码就会尽可能的合并到一个函数中去，也就变成了这样的类似代码

[
  /* 0 */
  function (module, exports, require) {
    //...
  }
]
这样的打包方式生成的代码明显比之前的少多了。如果在 Webpack4 中你希望开启这个功能，只需要启用 optimization.concatenateModules 就可以了。

module.exports = {
  optimization: {
    concatenateModules: true
  }
}
Tree Shaking
Tree Shaking 可以实现删除项目中未被引用的代码，比如

// test.js
export const a = 1
export const b = 2
// index.js
import { a } from './test.js'
对于以上情况，test 文件中的变量 b 如果没有在项目中使用到的话，就不会被打包到文件中。

如果你使用 Webpack 4 的话，开启生产环境就会自动启动这个优化功能。

小结
在这一章节中，我们学习了如何使用 Webpack 去进行性能优化以及如何减少打包时间。

Webpack 的版本更新很快，各个版本之间实现优化的方式可能都会有区别，所以我没有使用过多的代码去展示如何实现一个功能。这一章节的重点是学习到我们可以通过什么方式去优化，具体的代码实现可以查找具体版本对应的代码即可。








# 18.实现小型打包工具
原本小册计划中是没有这一章节的，Webpack 工作原理应该是上一章节包含的内容。但是考虑到既然讲到工作原理，必然需要讲解源码，但是 Webpack 的源码很难读，不结合源码干巴巴讲原理又没有什么价值。所以在这一章节中，我将会带大家来实现一个几十行的迷你打包工具，该工具可以实现以下两个功能

将 ES6 转换为 ES5
支持在 JS 文件中 import CSS 文件
通过这个工具的实现，大家可以理解到打包工具的原理到底是什么。

实现
因为涉及到 ES6 转 ES5，所以我们首先需要安装一些 Babel 相关的工具

yarn add babylon babel-traverse babel-core babel-preset-env  
接下来我们将这些工具引入文件中

const fs = require('fs')
const path = require('path')
const babylon = require('babylon')
const traverse = require('babel-traverse').default
const { transformFromAst } = require('babel-core')
首先，我们先来实现如何使用 Babel 转换代码

function readCode(filePath) {
  // 读取文件内容
  const content = fs.readFileSync(filePath, 'utf-8')
  // 生成 AST
  const ast = babylon.parse(content, {
    sourceType: 'module'
  })
  // 寻找当前文件的依赖关系
  const dependencies = []
  traverse(ast, {
    ImportDeclaration: ({ node }) => {
      dependencies.push(node.source.value)
    }
  })
  // 通过 AST 将代码转为 ES5
  const { code } = transformFromAst(ast, null, {
    presets: ['env']
  })
  return {
    filePath,
    dependencies,
    code
  }
}
首先我们传入一个文件路径参数，然后通过 fs 将文件中的内容读取出来
接下来我们通过 babylon 解析代码获取 AST，目的是为了分析代码中是否还引入了别的文件
通过 dependencies 来存储文件中的依赖，然后再将 AST 转换为 ES5 代码
最后函数返回了一个对象，对象中包含了当前文件路径、当前文件依赖和当前文件转换后的代码
接下来我们需要实现一个函数，这个函数的功能有以下几点

调用 readCode 函数，传入入口文件
分析入口文件的依赖
识别 JS 和 CSS 文件
function getDependencies(entry) {
  // 读取入口文件
  const entryObject = readCode(entry)
  const dependencies = [entryObject]
  // 遍历所有文件依赖关系
  for (const asset of dependencies) {
    // 获得文件目录
    const dirname = path.dirname(asset.filePath)
    // 遍历当前文件依赖关系
    asset.dependencies.forEach(relativePath => {
      // 获得绝对路径
      const absolutePath = path.join(dirname, relativePath)
      // CSS 文件逻辑就是将代码插入到 `style` 标签中
      if (/\.css$/.test(absolutePath)) {
        const content = fs.readFileSync(absolutePath, 'utf-8')
        const code = `
          const style = document.createElement('style')
          style.innerText = ${JSON.stringify(content).replace(/\\r\\n/g, '')}
          document.head.appendChild(style)
        `
        dependencies.push({
          filePath: absolutePath,
          relativePath,
          dependencies: [],
          code
        })
      } else {
        // JS 代码需要继续查找是否有依赖关系
        const child = readCode(absolutePath)
        child.relativePath = relativePath
        dependencies.push(child)
      }
    })
  }
  return dependencies
}
首先我们读取入口文件，然后创建一个数组，该数组的目的是存储代码中涉及到的所有文件
接下来我们遍历这个数组，一开始这个数组中只有入口文件，在遍历的过程中，如果入口文件有依赖其他的文件，那么就会被 push 到这个数组中
在遍历的过程中，我们先获得该文件对应的目录，然后遍历当前文件的依赖关系
在遍历当前文件依赖关系的过程中，首先生成依赖文件的绝对路径，然后判断当前文件是 CSS 文件还是 JS 文件
如果是 CSS 文件的话，我们就不能用 Babel 去编译了，只需要读取 CSS 文件中的代码，然后创建一个 style 标签，将代码插入进标签并且放入 head 中即可
如果是 JS 文件的话，我们还需要分析 JS 文件是否还有别的依赖关系
最后将读取文件后的对象 push 进数组中
现在我们已经获取到了所有的依赖文件，接下来就是实现打包的功能了

function bundle(dependencies, entry) {
  let modules = ''
  // 构建函数参数，生成的结构为
  // { './entry.js': function(module, exports, require) { 代码 } }
  dependencies.forEach(dep => {
    const filePath = dep.relativePath || entry
    modules += `'${filePath}': (
      function (module, exports, require) { ${dep.code} }
    ),`
  })
  // 构建 require 函数，目的是为了获取模块暴露出来的内容
  const result = `
    (function(modules) {
      function require(id) {
        const module = { exports : {} }
        modules[id](module, module.exports, require)
        return module.exports
      }
      require('${entry}')
    })({${modules}})
  `
  // 当生成的内容写入到文件中
  fs.writeFileSync('./bundle.js', result)
}
这段代码需要结合着 Babel 转换后的代码来看，这样大家就能理解为什么需要这样写了

// entry.js
var _a = require('./a.js')
var _a2 = _interopRequireDefault(_a)
function _interopRequireDefault(obj) {
    return obj && obj.__esModule ? obj : { default: obj }
}
console.log(_a2.default)
// a.js
Object.defineProperty(exports, '__esModule', {
    value: true
})
var a = 1
exports.default = a
Babel 将我们 ES6 的模块化代码转换为了 CommonJS（如果你不熟悉 CommonJS 的话，可以阅读这一章节中关于 模块化的知识点） 的代码，但是浏览器是不支持 CommonJS 的，所以如果这段代码需要在浏览器环境下运行的话，我们需要自己实现 CommonJS 相关的代码，这就是 bundle 函数做的大部分事情。

接下来我们再来逐行解析 bundle 函数

首先遍历所有依赖文件，构建出一个函数参数对象
对象的属性就是当前文件的相对路径，属性值是一个函数，函数体是当前文件下的代码，函数接受三个参数 module、exports、 require
module 参数对应 CommonJS 中的 module
exports 参数对应 CommonJS 中的 module.export
require 参数对应我们自己创建的 require 函数
接下来就是构造一个使用参数的函数了，函数做的事情很简单，就是内部创建一个 require 函数，然后调用 require(entry)，也就是 require('./entry.js')，这样就会从函数参数中找到 ./entry.js 对应的函数并执行，最后将导出的内容通过 module.export 的方式让外部获取到
最后再将打包出来的内容写入到单独的文件中
如果你对于上面的实现还有疑惑的话，可以阅读下打包后的部分简化代码

;(function(modules) {
  function require(id) {
    // 构造一个 CommonJS 导出代码
    const module = { exports: {} }
    // 去参数中获取文件对应的函数并执行
    modules[id](module, module.exports, require)
    return module.exports
  }
  require('./entry.js')
})({
  './entry.js': function(module, exports, require) {
    // 这里继续通过构造的 require 去找到 a.js 文件对应的函数
    var _a = require('./a.js')
    console.log(_a2.default)
  },
  './a.js': function(module, exports, require) {
    var a = 1
    // 将 require 函数中的变量 module 变成了这样的结构
    // module.exports = 1
    // 这样就能在外部取到导出的内容了
    exports.default = a
  }
  // 省略
})
小结
虽然实现这个工具只写了不到 100 行的代码，但是打包工具的核心原理就是这些了

找出入口文件所有的依赖关系
然后通过构建 CommonJS 代码来获取 exports 导出的内容
如果大家对于这个章节的内容存在疑问，欢迎在评论区与我互动。


# 19.React 和 Vue 两大框架之间的相爱相杀
React 和 Vue 应该是国内当下最火热的前端框架，当然 Angular 也是一个不错的框架，但是这个产品国内使用的人很少再加上我对 Angular 也不怎么熟悉，所以框架的章节中不会涉及到 Angular 的内容。

这一章节，我们将会来学习以下几个内容

MVVM 是什么
Virtual DOM 是什么
前端路由是如何跳转的
React 和 Vue 之间的区别
MVVM
涉及面试题：什么是 MVVM？比之 MVC 有什么区别？

首先先申明一点，不管是 React 还是 Vue，它们都不是 MVVM 框架，只是有借鉴 MVVM 的思路。文中拿 Vue 举例也是为了更好地理解 MVVM 的概念。

接下来先说下 View 和 Model：

View 很简单，就是用户看到的视图
Model 同样很简单，一般就是本地数据和数据库中的数据
基本上，我们写的产品就是通过接口从数据库中读取数据，然后将数据经过处理展现到用户看到的视图上。当然我们还可以从视图上读取用户的输入，然后又将用户的输入通过接口写入到数据库中。但是，如何将数据展示到视图上，然后又如何将用户的输入写入到数据中，不同的人就产生了不同的看法，从此出现了很多种架构设计。

传统的 MVC 架构通常是使用控制器更新模型，视图从模型中获取数据去渲染。当用户有输入时，会通过控制器去更新模型，并且通知视图进行更新。


MVC
但是 MVC 有一个巨大的缺陷就是控制器承担的责任太大了，随着项目愈加复杂，控制器中的代码会越来越臃肿，导致出现不利于维护的情况。

在 MVVM 架构中，引入了 ViewModel 的概念。ViewModel 只关心数据和业务的处理，不关心 View 如何处理数据，在这种情况下，View 和 Model 都可以独立出来，任何一方改变了也不一定需要改变另一方，并且可以将一些可复用的逻辑放在一个 ViewModel 中，让多个 View 复用这个 ViewModel。


以 Vue 框架来举例，ViewModel 就是组件的实例。View 就是模板，Model 的话在引入 Vuex 的情况下是完全可以和组件分离的。

除了以上三个部分，其实在 MVVM 中还引入了一个隐式的 Binder 层，实现了 View 和 ViewModel 的绑定。


同样以 Vue 框架来举例，这个隐式的 Binder 层就是 Vue 通过解析模板中的插值和指令从而实现 View 与 ViewModel 的绑定。

对于 MVVM 来说，其实最重要的并不是通过双向绑定或者其他的方式将 View 与 ViewModel 绑定起来，而是通过 ViewModel 将视图中的状态和用户的行为分离出一个抽象，这才是 MVVM 的精髓。

Virtual DOM
涉及面试题：什么是 Virtual DOM？为什么 Virtual DOM 比原生 DOM 快？

大家都知道操作 DOM 是很慢的，为什么慢的原因已经在「浏览器渲染原理」章节中说过，这里就不再赘述了。

那么相较于 DOM 来说，操作 JS 对象会快很多，并且我们也可以通过 JS 来模拟 DOM

const ul = {
  tag: 'ul',
  props: {
    class: 'list'
  },
  children: {
    tag: 'li',
    children: '1'
  }
}
上述代码对应的 DOM 就是

<ul class='list'>
  <li>1</li>
</ul>
那么既然 DOM 可以通过 JS 对象来模拟，反之也可以通过 JS 对象来渲染出对应的 DOM。当然了，通过 JS 来模拟 DOM 并且渲染对应的 DOM 只是第一步，难点在于如何判断新旧两个 JS 对象的最小差异并且实现局部更新 DOM。

首先 DOM 是一个多叉树的结构，如果需要完整的对比两颗树的差异，那么需要的时间复杂度会是 O(n ^ 3)，这个复杂度肯定是不能接受的。于是 React 团队优化了算法，实现了 O(n) 的复杂度来对比差异。 实现 O(n) 复杂度的关键就是只对比同层的节点，而不是跨层对比，这也是考虑到在实际业务中很少会去跨层的移动 DOM 元素。 所以判断差异的算法就分为了两步

首先从上至下，从左往右遍历对象，也就是树的深度遍历，这一步中会给每个节点添加索引，便于最后渲染差异
一旦节点有子元素，就去判断子元素是否有不同
在第一步算法中我们需要判断新旧节点的 tagName 是否相同，如果不相同的话就代表节点被替换了。如果没有更改 tagName 的话，就需要判断是否有子元素，有的话就进行第二步算法。

在第二步算法中，我们需要判断原本的列表中是否有节点被移除，在新的列表中需要判断是否有新的节点加入，还需要判断节点是否有移动。

举个例子来说，假设页面中只有一个列表，我们对列表中的元素进行了变更

// 假设这里模拟一个 ul，其中包含了 5 个 li
[1, 2, 3, 4, 5]
// 这里替换上面的 li
[1, 2, 5, 4]
从上述例子中，我们一眼就可以看出先前的 ul 中的第三个 li 被移除了，四五替换了位置。

那么在实际的算法中，我们如何去识别改动的是哪个节点呢？这就引入了 key 这个属性，想必大家在 Vue 或者 React 的列表中都用过这个属性。这个属性是用来给每一个节点打标志的，用于判断是否是同一个节点。

当然在判断以上差异的过程中，我们还需要判断节点的属性是否有变化等等。

当我们判断出以上的差异后，就可以把这些差异记录下来。当对比完两棵树以后，就可以通过差异去局部更新 DOM，实现性能的最优化。

当然了 Virtual DOM 提高性能是其中一个优势，其实最大的优势还是在于：

将 Virtual DOM 作为一个兼容层，让我们还能对接非 Web 端的系统，实现跨端开发。
同样的，通过 Virtual DOM 我们可以渲染到其他的平台，比如实现 SSR、同构渲染等等。
实现组件的高度抽象化
路由原理
涉及面试题：前端路由原理？两种实现方式有什么区别？

前端路由实现起来其实很简单，本质就是监听 URL 的变化，然后匹配路由规则，显示相应的页面，并且无须刷新页面。目前前端使用的路由就只有两种实现方式

Hash 模式
History 模式
Hash 模式
www.test.com/#/ 就是 Hash URL，当 # 后面的哈希值发生变化时，可以通过 hashchange 事件来监听到 URL 的变化，从而进行跳转页面，并且无论哈希值如何变化，服务端接收到的 URL 请求永远是 www.test.com。

window.addEventListener('hashchange', () => {
  // ... 具体逻辑
})
Hash 模式相对来说更简单，并且兼容性也更好。

History 模式
History 模式是 HTML5 新推出的功能，主要使用 history.pushState 和 history.replaceState 改变 URL。

通过 History 模式改变 URL 同样不会引起页面的刷新，只会更新浏览器的历史记录。

// 新增历史记录
history.pushState(stateObject, title, URL)
// 替换当前历史记录
history.replaceState(stateObject, title, URL)
当用户做出浏览器动作时，比如点击后退按钮时会触发 popState 事件

window.addEventListener('popstate', e => {
  // e.state 就是 pushState(stateObject) 中的 stateObject
  console.log(e.state)
})
两种模式对比
Hash 模式只可以更改 # 后面的内容，History 模式可以通过 API 设置任意的同源 URL
History 模式可以通过 API 添加任意类型的数据到历史记录中，Hash 模式只能更改哈希值，也就是字符串
Hash 模式无需后端配置，并且兼容性好。History 模式在用户手动输入地址或者刷新页面的时候会发起 URL 请求，后端需要配置 index.html 页面用于匹配不到静态资源的时候
Vue 和 React 之间的区别
Vue 的表单可以使用 v-model 支持双向绑定，相比于 React 来说开发上更加方便，当然了 v-model 其实就是个语法糖，本质上和 React 写表单的方式没什么区别。

改变数据方式不同，Vue 修改状态相比来说要简单许多，React 需要使用 setState 来改变状态，并且使用这个 API 也有一些坑点。并且 Vue 的底层使用了依赖追踪，页面更新渲染已经是最优的了，但是 React 还是需要用户手动去优化这方面的问题。

React 16以后，有些钩子函数会执行多次，这是因为引入 Fiber 的原因，这在后续的章节中会讲到。

React 需要使用 JSX，有一定的上手成本，并且需要一整套的工具链支持，但是完全可以通过 JS 来控制页面，更加的灵活。Vue 使用了模板语法，相比于 JSX 来说没有那么灵活，但是完全可以脱离工具链，通过直接编写 render 函数就能在浏览器中运行。

在生态上来说，两者其实没多大的差距，当然 React 的用户是远远高于 Vue 的。

在上手成本上来说，Vue 一开始的定位就是尽可能的降低前端开发的门槛，然而 React 更多的是去改变用户去接受它的概念和思想，相较于 Vue 来说上手成本略高。

小结
这一章节中我们学习了几大框架中的相似点，也对比了 React 和 Vue 之间的区别。其实我们可以发现，React 和 Vue 虽然是两个不同的框架，但是他们的底层原理都是很相似的，无非在上层堆砌了自己的概念上去。所以我们无需去对比到底哪个框架牛逼，引用尤大的一句话

说到底，就算你证明了 A 比 B 牛逼，也不意味着你或者你的项目就牛逼了... 比起争这个，不如多想想怎么让自己变得更牛逼吧。

