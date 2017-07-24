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

## 前端表现层基础

