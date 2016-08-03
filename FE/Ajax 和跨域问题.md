# Ajax和跨域问题

## XMLHttpRequest对象

- open():接受三个参数，发送请求名，url和是否异步请求的布尔值。
- readystatechange 事件监听，DOM0级方法，因为并非所有浏览器都支持 DOM2级方法。
- abort()方法可以取消异步请求，终止请求之后，还应该对 xhr 进行解引用操作，由于内存原因，不建议重用 xhr 对象。

 xhr.readyState状态码|说明
---|---
0|尚未初始化，还没有调用 open 方法
1|open()方法已调用，还未发送 send()方法
2|send()方法已调用，尚未接受响应
3|已经开始接受部分响应，但没有完全接收
4|响应已经接受完成

```javascript
var xhr = new XMLHttpRequest()
xhr.onreadystatechange = function() {
	if(xhr.readyState === 4) {
		if(xhr.status === 200 || xhr.status === 304) {
		   console.log(xhr.responseText)
		}
	}
}
xhr.open('GET', url, true)
xhr.send(null)
```

### HTTP 头部信息
- 使用 setRequestHeader()可以设置自定义的请求头部信息。接收两个参数，头部字段的名称和头部字段值。**必须在 open 方法后，send 方法前进行设置。**

```javascript
xhr.open('get', url, true)
xhr.setRequestHeader('myHeader', 'myValue') //open 和 send 中间设置
xhr.send(null)
```

- 调用 getResponseHeader()方法可以取得头部信息，接受一个包含头部字段名。
- 调用 getAllResponseHeaders()可以取得所有头部信息。是一个格式化过的长字符串。

```javascript
var header = xhr.getResponseHeader('myHeader') //'myValue'
var allHeaders = xhr.getAllResponseHeaders() //返回一个长字符串
```

### GET 请求
- 使用 get 请求通常会发生一个错误，就是查询字符串的格式有问题，一定要使用 encodeURIConponent()进行转码后在放到 url 末尾。
- 键值对以&分隔，url 跟？再接参数字符串。

```javascript
function addUrl(url, key, value) {  //添加参数的方法，向现有 url 末尾添加参数
	url += url.indexOf('?') == -1 ? '?' : '&'
	url += encodeURIComponent(name) + '=' + encodeURLIIComponent(value)
	return url
}

url = addUrl(url, 'username', 'www')
xhr.open('get', url, true)
```

### POST请求
- 默认情况下，服务器对 POST 请求不会和表单请求一视同仁，所以我们用 xhr 模仿表单提交，首先将头部 Content-type 信息设置为 application/x-www-form-urlencoded，也就是表单提交时的内容类型，然后以适当格式创建一个字符串。

```javascript
function submitData() {
	var xhr = new XMLHttpRequest()
	xhr.onreadystatechange = function(){...}
	
	xhr.open('post', url, true)
	
	xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded') //设置头部信息
	
	var form = document.getElementById('user-info')
	xhr.send(serialize(form)) //序列化表单数据的一个自定义方法
}
```

## XMLHTTPRequest2级
###FormData
- FormData 类型，为序列化表单以及创建与表单格式相同的数据提供便利。
- FormData类型有 append 方法，接受键值对这两个参数。

```javascript
var data = new FormData()
data.append('name', 'Nicholas') // FormData.append()方法

var data = new FormData(document.form[0]) //也可以直接传入一个表单元素
```

- 创建 FormData 实例后，可以传给 send 方法

```javascript
 xhr.open('post', url, true)
 var form = document.getElementById('user-info')
 xhr.send(new FormData(form))
```

## 进度事件
### load 事件
- 兼容性 IE8+。
- 只要浏览器接受响应，不管状态如何，都会触发 load 事件。所以还是得判断 status 状态码确定数据是否真的可用。

```javascript
var xhr = new XMLHttpRequest()
xhr.onload = function() { //实际只是少了 readyState判断，相比之下
	if(xhr.status === 200 || xhr.status === 304) {
		console.log(xhr.responseText)
	}
}
xhr.open('get', url, true)
xhr.send(null)
```

### progress 事件
- xhr.onprogress()事件会接收一个 event 对象，target 属性指向 xhr 对象，有三个额外的属性。

属性值|说明
---|---
event.lengthComputable|进度信息是否可用的布尔值
event.position|表示已经接收的字节数
event.totalSize|表示根据 Content-Length 响应头部确定的预期字节数

```javascript
var xhr = new XMLHttpRequest()
xhr.onprogress = function(e) { //进度事件
	if(e.lengthComputable) {
		var divProgress = document.getElementbyId('div')
		divProgress.innerHTML = e.position + '/' + e.totalSize
	}
}
xhr.onload = function() {...}
	//记得 progress 事件要在 open 之前调用，和 onload 一样
xhr.open(...)
xhr.send(null)
```

# 同源政策规避及跨域资源共享
## 同源政策
同源指三个方面：协议，端口，域名。三者全部相同才是同源。同源政策是安全基石，如果 Cookie 包含隐私，或者登录状态，假设 A 网站是银行， B 网站可以直接读取用户的 Cookie，那么用户隐私会被泄露，冒充登录等。**因为浏览器规定，表单提交不受同源政策限制。**

目前有三种行为受同源政策限制。

1. Cookie、LocalStorage 和 IndexDB 无法读取。
2. DOM 无法获得。
3. AJAX 请求无法发送。

### Cookie
Cookie 是服务器写入浏览器的一小段信息，只有同源网页才可以共享。但是如果两个网页一级域名相同，只有二级域名不同，浏览器允许通过设置 document.domain 共享 Cookie。

**需要注意的是，这种方法只适用 Cookie 和 iframe.** LocalStorage 和 IndexDB 需要使用下面的 PostMessage API.

```javascript
//假设 A 网页是 http://w1.baidu.com/a.html,B网页是 http://w2.baidu.com/b.html
//在 两个 网页下都设置
document.domain = 'baidu.com'

//在 A 网页下
document.cookie = 'test1=hello'
//在 B 网页下就可以读到这个 cookie
var cook = document.cookie
```
```javascript
//另外服务器端可以在设置 cookie 时指定 cookie 所属域名为一级域名，比如.baidu.com,这样二级域名三级域名都不需要任何设置就可以读取 cookie。
Set-Cookie: key=value; domain=.baidu.com;path=/
```

### iframe
如果两个窗口，比如 iframe 或者 window.open() 打开的窗口，一级域名相同，只是二级不同，可以设置 domain 规避同源政策拿到 DOM。
完全不同源的网站，目前有三种给你方法解决跨域窗口通信问题。

1. 片段识别符( fragment identifier)
2. window.name
3. 跨文档通信 API (Cross-document messaging)

#### 片段识别符
片段识别符就是 url 后面的#后面的部分。如果只是改变锚点，页面不会刷新。

```javascript
//父窗口写入子窗口
var src = url + '#' + data
document.getElementById('myIframe').src = src
//子窗口监听锚点改变
window.onhashchange = function() {
	var message = window.location.hash
}

//子窗口写入父窗口
parent.location.href = target + '#' + hash
```

#### window.name
浏览器有一个 window.name 属性，只要在一个标签内打开的网页是可以读取这个属性值的。
这个属性可以存很长容量的字符串，缺点是父窗口要监听 window.name 属性，影响页面性能。
使用方法：

1. 父窗口内打开非同源子窗口
2. 子窗口设置 window.name 属性
3. 子窗口跳转到同域的网址
4. 因为已经同源，父窗口现在可以读取子窗口 window.name 属性了

```javascript
window.name = data  //第二步 子窗口写入数据

window.location.href = 'http://parent.baidu.com/xxxx.html' //第三步跳转到同源网址

//在父窗口下读取数据
var data = document.getElementById('myIframe').contentWindow.name
```

#### window.postMessage
上述两种是破解方法，html5提供了原生的夸文档消息传递，简称 XDM。兼容性IE8+。
既稳妥又简单的实现跨文档通信，比如 www.baidu.com 向 内嵌的一个p2p.baidu.com 页面进行通信。
核心是 postMessage()方法，接受两个字符串参数，传递的参数字符串和接收消息的窗口源，如果设置为*代表任意窗口都可以接收。第二个参数非常重要，防止发送到不安全的地方。

message事件参数：

1. e.data:传递的参数字符串(虽然不一定是字符串，但是最好还是用 JSON.stringify()方法转成字符串再传递)
2. e.origin:消息发到了哪个域
3. e.source: 发送消息的的window 对象的代理，并非真是 window对象，所以只使用 postMessage() 方法就好

```javascript
//子窗口向父窗口传递信息
var iframeWindow = document.getElementById('myIframe').contentWindow
iframeWindow.postMessage('hello world', 'http:// www.baidu.com')

//父窗口消息监听 message
window.onmessage = function(e) {
	if(e.origin === 'http://www.baidu.com') { //确保消息传递的是给自己的，过滤不属于本窗口的信息
		processMessage(e.data)
		e.source.postMessage('received', 'http://p2p.baidu.com') //给子窗口发送回执
	}
}
```

#### LocalStorage
这个通过 window.postMessage()方法，父子窗口可以相互读写自己的 localStorage.

```javascript
//父窗口写入子窗口iframe的localStorage
//父窗口发送信息
var myIframe = document.getElementById('iframe').contentWindow
var obj = { name: 'Jack' }
myIframe.postMessage(JSON.stringify(obj), 'http://p2p.baidu.com')

//子窗口监听 message 事件获取信息写入 localStorage
myIframe.onmessage = function(e) {
	if(e.origin === 'http://p2p.baidu.com') {
		var data = JSON.parse(e.data)
		localStorage.setItem(data.key, data.value)
	}
}
```


# AJAX 请求规避同源限制
同源政策规定，ajax请求只能发给同源网址，否则就报错。
除了架设服务器代理（浏览器请求同源服务器，再由后者请求外部服务），有三种方法规避这个限制。

1. JSONP
2. WebSocket
3. CORS

## JSONP
简单使用，浏览器兼容性极佳，但是只能发 GET 请求。
它的基本思想是，网页通过添加一个`<script>`元素，向服务器请求JSON数据，这种做法不受同源政策限制；服务器收到请求后，将数据放在一个指定名字的回调函数里传回来。

```javascript
function addScriptTag(src) { //动态增加scirpt
	var script = document.createElement('script')
	script.setAttribute('type', 'text/javascript')
	script.src = src
	document.body.appendChild(script)
}

window.onload = function() {  //网页加载后调用方法增加 script 标签
	addScriptTag('http://www.baidu.com/heiheihei.php?callback=foo')
}

function foo(data) { //回调函数，只要浏览器定义了这个方法，请求返回后立即调用
	console.log(data.ip)
}
```

## WebSocket
