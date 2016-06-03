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

