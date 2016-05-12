# BOM对象
## window对象
- BOM的核心是window，它表示浏览器的一个实例。window 对象既是通过 js 访问浏览器窗口的一个接口，又是 es 规定的Global 对象。
- 全局变量不能通过 delete 操作符删除，但是 window 对象上的属性是可以的。
- 直接访问为未声明的变量会报错，但是可以通过 window对象上的属性来查询某个未声明的变量是否存在。

```javascript
var newValue = oldValue  //会抛出错误，因为后者未声明
var newValue = window.oldValue  // undefined
```

## 间歇调用和超时调用
- JS 是单线程语言，但它允许通过设置超时值和间歇时间值来调度代码在特定的时刻执行。前者是在指定的时间过后执行代码，后者则是每隔指定的时间就执行一次。
- `setTimeout()`方法可以接受两个参数，第一个是要执行的代码，也可以是字符串，但是推荐用`function（）{}`。后一个参数是执行前需要等待的毫秒数。传递字符串可能导致性能损失，因此不建议。
- 值得注意的是，经过了指定的毫秒，代码也不一定会执行。js 是一个单线程序的解释器，有一个任务队列。如果队列是空的，那么代码会立即执行；如果队列不是空的，那么它就要等前面的代码执行完毕以后再执行。
- 取消的时候可以用` clearTime()`方法.把相应的数字 id 传递进去。
- 注意在以上方法中this 指向全局，window 对象。严格模式下是 undefined。

```javascript
// timeoutId 是一个数字 ID
var timeoutId = setTimeout(function() {
	console.log('heiheihei')
}, 1000)

clearTimeout(timeoutId) //即立即取消这个超时调用
```

- 取消间隔调用` setInterval()`远比取消` setTimeout()`重要的多，因为间隔调用如果不干涉会一直执行。
- 一般认为，使用setTimeout 模拟 setInterval 是最佳实践。因为后一个间歇调用可能会在前一个间歇调用结束前启动。使用模拟方式则可以避免这一情况。

```javascript
var num = 0
var max = 10

function inc() { //模拟setInterval 方法，间隔增加直至最大值
	num++
	
	if (num < max) {
		setTimeout(inc, 500)
	} else {
		console.log('done')
	}
}

setTimeout(inc, 500)
```

## location对象
- location 是最有用的 BOM 对象之一，而且它既是 window 对象的属性，也是 document 对象的属性。 `window.location`和` document.location`引用的是同一个对象。
- location 对象的用处不止表现在它保存着当前文档的信息，而且还可以通过不同属性访问 URL 的解析片段。

	属性名 | 例子 | 说明
	----|:------:|----
	hash | '#contents'  | URL 中的 hash，#后面的字符，没有则为空
	host | 'www.baidu.com:80'  | 服务器名称和端口号（如果有）
	hostname | 'www.baidu.com'  | 服务器名称
	href | 'www.baidu.com:80/dd.html#hash1?q=name'  | 完整URL
	pathname | '/dd.html'  | URL 中的目录和文件名
	port|'80'|端口
	protocol| 'http:'|协议名和冒号
	search|'?q=name'|查询字符串，别忘了有问号开头
	
	
- 尽管 `location.search` 可以查询到查询的字符串，但是没法逐个访问，并不方便。创建一个函数返回对象，得到对应的 key 和 value。

```javascript
  function getQueryStringArgs() {
  	//字符串是否为空
    var qs = (location.search.length > 0 ? location.search.substring(1) : '')
    var args = {}
    var items = qs.length ? qs.split('&') : []
    var item = null
    var name = null
    var value = null

    for (var i = 0; i < items.length; i++) {
      item = items[i].split('=')
      //进行解码
      name = decodeURIComponent(item[0])
      value = decodeURIComponent(item[1])

      if (name.length) {
        args[name] = value
      }
    }
    return args
  }
```

### 位置操作location

```javascript
		//修改 URL
location.assign('http://www.baidu.com')
window.location = 'xxx'
location.href = 'xxx'
```

- 上述的三种方式得到的效果完全一样，因为后面的两个也会调用assign()方法。

```javascript
//将 URL 修改成'http://www.baidu.com#section1' 页面不跳转
location.hash = '#section1'

//将 url 修改成'http://www.baidu.com:80' 页面跳转
location.port = 80
```

- 每次修改 location 的属性（hash 除外），页面都会以新 URL 重新加载。
- 当通过上述任何一种方式修改 URL 后，浏览器的历史记录就会生成一条新纪录。通过后退按钮都会回到前一个页面。
- 使用 replace()方法可以禁用后退按钮。

```javascript
//浏览器跳转到百度，但不会在历史纪录中生成新纪录，而且不能回到之前的页面
location.replace('http://www.baidu.com')
```

- `location.reload()`方法会让浏览器以最有效的方式重新加载，可能使用缓存。
- `location.reload(true)`方法会让浏览器强制完全重载。
- reload()方法之后的代码**可能，也可能不会执行**，所以放在最后一行吧。

##history 对象

```javascript
history.go(-1)	//后退一页
history.go(1)		//前进一页
history.go(2)		//前进2页

history.go('www.baidu.com') //跳转到最近的百度页面，如果没有这条字符串，那么什么也不做

history.back()
history.forwward()	//前进后退的简写方法

history.length  //数量，新加载的页面是0
```