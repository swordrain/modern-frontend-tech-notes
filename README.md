# 《现代前端技术解析》笔记

## Web前端技术基础
**现代Web前端技术概述**

* 借助库和框架
* 组件化思想
* 异步
* 图片优化处理（如webp格式）
* 合理的缓存
* 移动端、桌面端区别对待

**Web前端技术的发展**

1. 静态黄页
2. 服务器组装动态网页数据
3. 后端为主的MVC
4. 前后端分离
5. 纯前端MV*为主、中间层直出
6. 前端Virtual DOM、MNV*、前后端同构

**浏览器处理步骤**

1. 接收到用户输入网址后，开启线程，按照不同的协议进行处理
2. 调用浏览器引擎中对应的方法，分析并加载URL
3. 通过DNS解析获取网站的IP，查询完成后连同浏览器的Cookie、userAgent等信息向网站目的IP发出GET请求
4. 进行HTTP协议会话，浏览器向服务器发送报文
5. 进入网站后台上的Web服务器处理请求，如Apache、Tomcat等
6. 进入部署好的后端应用，如PHP、Java等，找到对应的处理逻辑
7. 服务器处理请求并返回响应报文，如果命中缓存，返回304，否则返回200
8. 浏览器下载HTML文档或从本地缓存读取文件内容
9. 浏览器根据HTML文件解析结构并建立DOM文档树，并根据HTML中的标记请求下载指定的MIME类型文件（CSS、js脚本等），同时设置缓存等内容
10. 页面开始解析渲染DOM，CSS根据规则解析并结合DOM文档树进行网页内容布局和绘制渲染，JS根据DOM API操作DOM，并读取浏览器缓存、执行事件绑定等，整个页面展示过程完成

**浏览器的组成部分**

1. 用户界面
2. 网络
3. JS引擎
4. 渲染引擎
5. UI后端
6. JS解释器
7. 持久化数据存储

**渲染引擎的工作流程**

1. 解析HTML构建DOM树
2. 构建渲染树
3. 渲染树布局
4. 绘制渲染树

**数据持久化存储技术**

1. HTTP文件缓存（浏览器缓存）
2. localStorage
3. sessionStorage
4. Cookie（HttpOnly的Cookie不能被document.cookie获取并修改）
5. WebSQL
6. IndexDB
7. Application Cache（manifest配置文件）
8. cacheStorage（在ServiceWorker规范中定义，用于保存每个ServiceWorker声明的Cache对象，未来代替Application Cache）
9. Flash缓存

**开发工具与调试工具**

* sublime
* webstorm
* vscode
* vim
* Chrome Develop Tool
* Fiddler
* node-inspector
* Vorlon.js(For mobile)
* Weinre(For mobile)


## 前端与协议
### HTTP
HTTP报文由头部、空行、正文三部分组成，空行用于区分头部和正文，由一个回车符和一个换行符组成

请求头一般包括

* 请求类型
* 请求URI
* 协议版本
* 扩展内容
* 请求头部域（Accept Cookie Cache-Control Host等）
* 空行
* 正文内容

响应头一般包括

* 状态码
* 状态描述
* 协议版本
* 扩展内容
* 响应头部域（Date Content-Type Cache-Control Expire等）
* 空行
* 正文内容

**HTTP1.1的增强**

* 长连接（Connection: keep-alive，连接完成不会立即断开）
* 协议扩展切换
* 缓存控制（Cache-Control）
* 部分内容传输优化

**HTTP2的增强**

* 基于SPDY2协议
* 二进制传输（HTTP 1.x默认基于文本格式）
* TCP多路复用降低网络请求连接建立和关闭的开销
* 支持传输流的优先级和流量控制机制
* 支持服务器端推送

### Web安全机制

* XSS
* SQL注入
* CSRF
* 请求劫持（DNS劫持、HTTP劫持）

**HTTPS协议通信过程**

HTTPS加入SSL层加密HTTP数据，启用默认为443的端口进行数据传输

![https](https://github.com/swordrain/modern-frontend-tech-notes/blob/master/https.png)

**浏览器Web安全控制**

一些特定的header头配置

* X-XSS-Protection
* Strict-Transport-Security
* Content-Secutiry-Policy
* Access-Contorl-Allow-Origin

### 前端实时协议

* WebSocket
* Poll
* Long Poll
* DDP（Distributed Data Protocol分布式数据协议）- Node.js可通过ddp模块使用

### Restful数据协议规范
满大街了，略

### 与Native交互协议
#### Web到Native协议调用
**通过URI请求**

Native应用在系统注册一个Scheme协议的URI，这个URI可在系统的任意地方授权访问原生代码。Native的WebView控件中的JS脚本请求也可以匹配该通用的Scheme协议

```
let iframe = document.createElement('iframe')
iframe.setAttribute('style', 'display: none')
document.body.appendChild(iframe)
iframe.setAttribute('src', 'myApp://className/method?args')
```

**通过addJavascriptInterface注入方法到页面中调用**

Java代码侧

```
WebSettings webSettings = webView.getSettings();
webSettings.setJavaScriptEnabled(true);
webView.loadUrl("file:///android_asset/index.html");
webView.addJavascriptInterface(new JsInterface(), "native"); // webView里有个全局变量叫native

public class JsInterface(){
	@JavascriptInterface
	public void showToast(String toast){
		Toast.makeText(MainActivity.this, toast, Toast.LENGTH_SHORT).show();
	}
}
```

JS代码侧

```
<script>
	function nativeAlert(msg){
		native.showToast(msg);
	}
</script>
```

#### Native到Web协议调用
通过Native的`loadUrl`(Android)或`stringByEvaluatingJavaScriptFromString`(iOS)方法实现

```
WebSettings webSettings = webView.getSettings();
webSettings.setJavaScriptEnabled(true);
webView.loadUrl("file:///android_asset/index.html");
JsInterface jsInterface = new JsInterface();
jsInterface.log('hello');

public class JsInterface{
	public void log(final String msg){
		webView.post(new Runnable() {
			@Override
			public void run(){
				webView.loadUrl("javascript: log(" + "'" + msg + "'" + ")")
			}
		})
	}
}
```

JS侧有个log方法

```
<script>
	function log(msg){
		console.log(msg)
	}
</script>
```

另一种替代方案是使用`setWebChromeClient`方法，它可以重写`onJsAlert`和`onJsPrompt`方法来监听webView中的`alert`和`prompt`方法

```
webView.setWebChromeClient(new WebChromeClient(){
	@Override
	public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result){
		result.confirm(JSBridge.callJsPrompt(MainActivity.this, view, message));
		return true;
	}
})
```

一个常用的通信规则是`jsbridge://className:callbackMethod/methodName?jsonObj`，`callbackMethod`是为了在JS调用Native方法成功后异步通过`loadUrl('javascript:callbackMethod()')`来让JS继续进行操作，给一个例子`jsbridge://Util:success/toString?{"msg": "hello"}`

一段通用的注册代码

```
JSBridge.call = function(className, methodName, params, callback){
	let bridgeString
	let paramsString = JSON.stringify(params || {})
	if(className && methodName){
		bridgeString = `jsbridge://${className}:${callback}/${methodName}??${paramsString}`
		try{
			sendToNative(bridgeString)	
		}catch(e){
			console.log(e)
		}
	} else {
		console.log('invalide className or methodName')
	}
}

funciton sendToNative(uri, data){
	window.prompt(uri, JSON.stringify(data || {}))
}
```

## 前端三层结构与应用

### HTML结构层基础

* DOCTYPE的演变
* 标签要语义化
* Google的AMP HTML提议规范
	* 只允许异步的script脚本
	* 只加载静态的资源
	* 不能让内容阻塞渲染
	* 不在关键路径中加载第三方JS
	* 所有的CSS必须内联
	* 字体使用声明必须高效
	* 最小化样式声明
	* 只运行GPU加速的动画
	* 处理好资源加载顺序问题
	* 页面必须立即加载
	* 提升AMP元素性能

### 前端结构层演进

* XML与HTML
* HTML5标准
* Web Component与Shadow DOM

### 浏览器脚本演进

* CoffeeScript
* ECMAScribe标准
* TypeScript
* JS衍生脚本（JSX等）

### JS标准实践

#### ES5

* 严格模式
* JSON对象
* Object对象增强
* Array对象增强
* Funtion方法bind
* String方法trim
* Date方法now、toJSON

#### ES6

* let const
* 字符串模板
* 解构
* Array增强
* 函数参数增强
* 箭头函数
* Object增强
* class
* module
* Iterator
* Generator
* Map Set WeakMap WeakSet
* Promise
* Symbol
* Proxy
* ...

### 前端表现层基础

* CSS选择器与优先级
* CSS属性
	* 布局类
	* 几何类
	* 文本类
	* 动画类
	* 查询类

### 前端界面技术

CSS样式统一化：reset normalize neat

CSS预处理： SASS LESS Stylus postCSS

动画：

* JS实现
* SVG
* CSS3 transition
* CSS3 animation
* Canvas
* requestAnimationFrame

### 响应式网站

1. 根据userAgent发送302响应跳转
2. 使用JS、media query、<picture>、rem等手段

## 现代前端交互框架

**直接操作DOM**

* 原生DOM方法 
* jQuery

**MV*交互模式**

* MVC 
* MVP 
* MVVM
* Virtual DOM

**MVVM的数据变更检测方法**

* 手动触发绑定
* 脏值检测
* 前端数据对象劫持（使用Object.defineProperty、Object.defineProperties对ViewModel对象进行get()和set()的监听）
* ES6 Proxy

**MNV*时代**

Model-NativeView-* 如React-Native解决方案，转为原生控件展示

## 前端项目与技术实践
### 前端开发规范
**前端通用规范**

* HTML CSS JS结构分离
* 统一用tab缩进
* 用`<meta charset="utf-8">`指定编码
* 一般都用小写
* 代码单行长度限制（120或80字符）
* 注释
* 删除行尾空格与符号

**前端HTML规范**

* 用`<!DOCTYPE html>`定义文档类型
* head内容包含title、keyword、description等SEO所需要的内容
* 省略type，如`text/css`、`text/javascript`
* 使用双引号包裹属性值
* 属性值省略，不用指定`true`或`readonly`等
* 元素正确的嵌套，如inline元素里不嵌套block元素
* 标签合理闭合
* 使用img的alt属性
* 使用label的for属性
* 按模块添加注释，如`<!-- News List module -->`
* 元素标签合理的分行、缩进
* 标签语义化

**前端CSS规范**

* 引用规范，不推荐style
* 合理命名，class用中划线-，样式一般不用id（没法复用）
* 简写方式，0不需要单位，URL资源的引号不需要
* 属性书写顺序，先布局后内容
* Hack写法，先私有属性后标准属性（-webkit-box-shadow在box-shadow之前）
* 高效实现，不冗余
* 使用预处理脚本

**ES5规范**

* 分号统一加上
* 合理空格、空行
* 字符串最外层用单引号
* 合理变量命名，jQuery对象用$开头
* 对象属性名不加引号，键值缩进，数组、对象属性后不能有逗号
* 代码块大括号不省略
* 条件判断要严谨
* 不在条件语句或循环语句中声明函数

**ES6规范**

* 正确使用let const
* 字符串拼接使用字符串模板
* 解构赋值尽量就一层
* 数组拷贝推荐使用...实现
* 数组遍历推荐用for of
* 尽量使用class定义类，使用constructor进行属性成员变量赋值
* 模块化多变量导出尽量使用对象解构，不适用全局导出，不把import和export写在同一行
* 导出类名时，保持模块名称和文件名相同，首字符大写
* yield进行异步操作时用try catch包括，方便对异常处理
* 推荐使用Promise而不是第三方库
* 避免使用迭代器
* 合理使用Generator，推荐使用async/await

### 前端组件规范
**UI组件规范**

**模块化规范**

* AMD
* CMD
* CommonJS
* ES6 module

**项目组件化设计规范**

* Web Component组件化
* MVVM框架组件化
* Virtual DOM组件化
* 基于目录管理的通用组件化

### 自动化构建

构建步骤

1. 读取入口文件
2. 分析模块引用
3. 按照引用记载模块
4. 模块文件编译处理
5. 模块文件合并
6. 文件优化处理
7. 写入生成目录

构建解决问题

1. 模块分析引入
2. 模块化规范支持
3. CSS编译、自动合并图片（CSS sprite）
4. 压缩优化
5. HTML路径分析替换
6. 区分开发和上线目录环境
7. 异步文件打包方案
8. 文件目录白名单设置

### 前端性能优化
#### 前端性能测试
**Performance Timing API**

![Performance](http://images2015.cnblogs.com/blog/595796/201603/595796-20160323172552386-842768536.png)

[详情戳这里](https://www.w3.org/TR/resource-timing/)

**Profile工具**

使用`console.profile()`和`console.profileEnd()`配合浏览器开发工具中的Profile

**页面埋点计时**

**资源加载时序图**

#### 桌面浏览器前端优化策略

* 网络加载
	* 减少HTTP请求次数
	* 减小HTTO请求大小
	* 将CSS和JS放在外部文件以缓存
	* 避免页面中空的href和src（渲染时仍会加载，直至失败）
	* 指定合理的缓存
	* 减少重定向
	* 使用静态资源分域存放来增加下载并行数（一个域名的下载并行是有限的）
	* 使用静态资源CDN存储文件
	* 使用CDN Combo下载传输内容
	* 使用可缓存的AJAX
	* 使用GET请求AJAX
	* 减小Cookie的大小并进行Cookie隔离
	* 减小favicon.ico并缓存
	* 推荐使用异步JS资源
	* 消除阻塞渲染的CSS及JS
	* 避免使用CSS import
* 页面渲染
	* 把CSS资源引用放到HTML顶部
	* JS资源放到HTML底部
	* 不在HTML中直接缩放图片（引起reLayout和reRender）
	* 减少DOM元素数量和深度
	* 避免使用`<table>`、`<iframe>`等慢元素（全部渲染完后才一次性绘制到页面上）
	* 避免耗时的JS
	* 避免CSS表达式或滤镜

#### 移动端浏览器前端优化策略

* 网络加载
	* 首屏数据请求提前，避免JS文件加载后才请求数据
	* 首屏加载和按需加载，非首屏内容滚屏加载，保证首屏内容最小化
	* 模块化资源并行下载
	* inline首屏必备的CSS和JS（写在HTML里）
	* meta dns prefetch设置DNS预解析
	* 资源预加载
	* 合理利用MTU策略
* 缓存
	* 合理利用浏览器缓存
	* 静态资源离线方案
	* 尝试使用AMP HTML
* 图片
	* 图片压缩处理
	* 使用较小的图片或Base64编码的图片
	* 使用更高压缩比的图片
	* 图片懒加载
	* 使用Media Query或srcset按需加载
	* 使用iconfont代替图片图标
	* 定义上传图片大小限制
* 脚本
	* 尽量使用id选择器（快）
	* 合理缓存DOM对象
	* 页面元素尽量使用事件代理，避免直接事件绑定
	* 使用touchstart代替click
	* 避免touchmove、scroll连续事件处理（或设置合理节流）
	* 避免使用eval、with，使用join代替+，推荐使用字符串模板
	* 尽量使用ES6特性
* 渲染
	* 使用Viewport固定屏幕渲染，可以加速页面渲染内容
	* 避免各种形式重排重绘
	* 使用CSS3动画，开启GPU加速（`transform: translateZ(0)`）
	* 合理使用Canvas和requestAnimationFrame
	* SVG代替图片
	* 不滥用float
	* 不滥用web字体或过多font-size声明
* 架构协议
	* 尝试使用SPDY和HTTP2
	* 使用后端渲染数据
	* 使用Native View代替DOM

### 前端用户数据分析
#### 用户访问统计

* PV(Page View)
* UV(Unique Visitor) - 可以基于1.cookie和IP或2.userAgent和IP进行统计
* VV(Visit View)
* IP(访问站点的不同IP数)

#### 用户行为分析

* 页面点击量
* 用户点击流分析
* 用户访问路径分析
* 用户点击热力图
* 用户转化率与导流转化率
* 用户访问时长、内容分析

#### 前端日志上报

**错误日志获取**

* try catch
	* 捕捉运行时错误
	* 无法捕捉语法错误
	* 如果是异步函数的内容，要把function函数块内容全部加入到try catch中执行
* window.onerror
	* 可以在任何上下文中执行
	* 一般捕捉语法错误和运行时错误
	* 如果JS和HTML不在同一个域名下，出错时errorMsg全部为script error而不是具体错误描述信息，此时要添加JS脚本跨域设置`<script src="main.js" crossorigin></script>`

**错误信息上传**

**高效查找问题**

**文件加载失败监控**

**性能分析上报**

### 前端搜索引擎优化基础

* title、keywors、description的优化
* 语义化标签的优化
* URL规范化
* robot.txt文件引导爬虫
* sitemap.html或sitemap.xml

## 前端跨栈技术

**前后端同构**

**Hybrid技术**

## 未来前端时代
### 未来前端趋势

* 新标准的进化与稳定
* 应用开发技术趋于稳定并等待下一次革新
* 持续不断的技术工具探索
* 浏览器平台新特性
* 更优化的前端技术开发生态
* 前端新领域的出现 - VR、IoT

### 做一名优秀的前端工程师

* 学会高效沟通
* 使用高效的开发工具
* 处理问题方法
* 学会前端项目开发流程设计
* 持续的知识和经验积累管理
* 切忌过分追求技术
* 必要的产品设计思维


