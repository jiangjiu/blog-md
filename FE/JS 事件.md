# 事件
## 冒泡和捕获阶段

![](http://www.admin10000.com/UploadFiles/Document/201503/21/20150321132128500929.JPG)

- 一图流解释：IE 事件流叫做事件冒泡，从元素逐级向上传递，所有现代浏览器支持事件冒泡。
- 事件捕获是网景团队提出，虽然规范要求应该从 document 对象开始，但浏览器基本都从 window 对象开始。特殊需求才会使用。

## 目标阶段
- 需要注意的是，你以为点击蚊子，事件目标节点在div上，其实实际触发会在最深的节点，比如 p 或者 span 等子节点上。

## DOM 事件流
- *DOM2级事件*规定的事件流包括三个阶段：事件捕获，目标事件阶段和事件冒泡阶段。
- 规范要求：捕获阶段不触发目标元素事件，然后目标事件处理，再进行冒泡阶段。
- 然而多数浏览器在捕获阶段也实现了目标元素事件，导致有两次机会可以实现目标元素事件。IE9+。

## 监听事件
### HTML 内联属性（避免使用）
- HTML 元素里面直接填写事件有关属性，属性值为 JavaScript 代码，即可在触发该事件的时候，执行属性值的内容。
- onclick 属性表示触发 click，属性值的内容（JavaScript 代码）会在单击该 HTML 节点时执行。
- 显而易见，使用这种方法，JavaScript 代码与 HTML 代码耦合在了一起，不便于维护和开发。所以除非在必须使用的情况（例如统计链接点击数据）下，尽量避免使用这种方法。

```javascript
<button onclick='alert('hhh')'>点击</button>
```

### DOM 属性绑定
- 也可以直接设置 DOM 属性来指定某个事件处理的函数。
- 上面代码就是监听 element 节点的 click 事件。它比较简单易懂，而且有较好的兼容性。但是也有缺陷，因为直接赋值给对应属性，如果你在后面代码中再次为 element 绑定一个回调函数，会覆盖掉之前回调函数的内容。
- 虽然也可以用一些方法实现多个绑定，但还是推荐标准事件监听函数。
- 这种方式添加的事件处理程序会在事件的冒泡阶段处理，同时 this 引用当前元素。

```javascript
var btn = document.getElementById('button')
btn.onclick = function() {
	alert('ddd')
}	//属性绑定事件

btn.onclick = null   //删除事件处理程序  HTML 绑定事件也可以这样解除
```

### 事件监听函数
- addEventListener()和 removeEventListener()两个方法，所有 DOM 节点都包括这两个方法。
- 接受三个参数，事件名，处理函数，一个布尔值。

布尔值|说明
---|---
true|捕获阶段
false|冒泡阶段  常用

```javascript
btn.addEventListerner('click', function(){
	alert('ddd')
}, false)

btn.addEventListerner('click', handler, false)
btn.removeEventListener('click', handler, false) //移除监听事件，注意必须是同一引用 handler
```

- 和上面的属性绑定事件相同，this 指向绑定的元素，false 是冒泡阶段触发，这也是最常用的，因为要兼容 IE。
- 最后移除事件监听只可以用 removeEventListener()方法，同时**必须是同一引用函数，使用匿名函数，即使完全相同也是不可以的，因为并非同一引用**。

## 事件对象
- 在触发 DOM 上的某个事件时，会产生一个事件对象 event, 对象中包含这所有与事件相关的信息。
- 在事件处理程序内部，this 始终等于 currentTarget 的值，而 target 的值则只包含实际目标。如果直接属性绑定，this、currentTarget 和 target 指向目标，三者值相同。
- 如果事件处理程序存在于按钮的父节点，target 指向目标，currentTarget 和 this 指向父节点。

```javascript
var btn = document.querySelector('#btn')
btn.onclick = function(e){
	alert(e.currentTarget === e.target)  //true
	alert(e.target === this)	//true 属性绑定，三者相同
}

document.body.onclick = function(e) {
	alert(e.currentTarget === e.target) //false
	alert(e.currentTarget === this)  //true this 指向 body
	alert(e.target === btn) //true 点击 btn 按钮，target 属性指向 btn
}
```

- 在需要通过一个函数处理多个时间时，可以使用 e.type 属性，定义一个函数，处理多种事件。

```javascript
var handler = function(e) {
	switch (e.type) {
		case 'click' :
			alert('clicked')
			break
		
		case 'mouseover':
			e.target.style.color = 'red'
			break
			
		case 'mouseout':
			e.target.style.color = 'black'
			break
			
	}
}

btn.onclick = handler
btn.onmouseover = handler
btn.onmouseout = handler
```

- 阻止默认行为可以用 e.preventDefault()方法；只有 cancelable 属性为 true 的事件，才可以用 preventDefault()阻止默认事件。
- e.stopPropagation()方法用于取消进一步的事件捕获或冒泡。例如注册在 btn 上的事件处理程序立即调用 e.stopPropagation()方法，从而避免触发注册在 document.body 上的事件。防止出现两次或者重叠的情况。
- 事件对象的 e.eventPhase 属性，用于确定正位于事件流的那个阶段。

e.evenetPhase属性值|说明
---|---
1|捕获阶段
2|事件处理程序正处于目标对象上
3|冒泡阶段
0|none
- 只有事件处理程序执行期间，event 对象才会存在，一旦事件处理程序执行完成，event 对象就会销毁。

**下面列出一些 event 对象的属性值。**

属性|说明
---|:---:
e.type   **string**|事件的名称，比如'click'
e.target **node**|事件要触发的目标节点
e.bubbles **boolean**|表明改时间是否在冒泡阶段触发
e.preventDefault() |禁止默认事件
e.stopPropagation()|停止进一步冒泡或捕获阶段
e.eventPhase **number**|见上
e.pageX和 e.pageY **number**|表示触发事件时，鼠标相对于页面的坐标
e.isTrusted **boolean**|浏览器触发（用户真实操作触发），还是 js 代码触发。

## 提升性能
在 js 中，添加到页面上的事件处理程序数量将直接关系到页面运行性能。

- 每个函数都是对象，都会占用内存；内存中的对象越多，性能就越差。
- 必须事先指定所有事件处理程序而导致的 DOM 访问次数，会延迟整个页面的交互就绪时间。

### 事件委托
- 事件委托利用了事件冒泡的机制，例如 click 事件会一直冒泡到 document 层次，只要给整个页面设置一个监听，就不不必分开一一设置了。

```javascript
var div = document.querySelector('#myDiv')
div.addEventListener('click', handler, false)

function handler(e) {
	var target = e.target
	
	switch(target.id) {
		case btn:
			alert('btn is clicked!')
			break
		case li:
			alert('li is clicked!')
			break
	}
}
```

上述代码把 div 中的 btn 和 li 元素没有分开监听，给父元素设置lisener 同时根据元素的 id 进行切换，这就是初步的事件代理。

- 如果可行的话，给 document 对象设置一个事件处理程序，泳衣处理页面上的某种特定类型的事件。
- 优点是，document 对象很快就可以访问，无需等待 DOMContentLoaded 或者 load 事件，只要可单击的元素成现在页面上，立即就可以具备适当的功能。
- 在页面中设置事件处理程序所需的事件更少，只添加一个监听事件，DOM 引用更少，花的时间也少。
- 整个页面占用的内存少，能够提升整体性能。

最合适采用事件委托技术的事件包括 click、mousedown、mouseup、keydown、keyup和 keypress。虽然 mouseover 和 mouseout 事件也冒泡，但要适当处理很不容易，需要经常计算元素的位置。

### 移除监听事件
- 文档中移除带有事件监听的元素时，removeChild(), replaceChild()，innerHTML 这些方法，很可能监听事件无法被当做垃圾回收。
- 再有就是卸载页面的时候，如果没有清理干净监听事件，就可能会只留在内存中。最好的做法是在页面卸载前，通过 onunload 事件移除所有监听事件，事件代理在此极具优势，因为监听事件很少。

```javascript
btn.onclick = null  //设置为 null  移除监听事件
```

## 自定义事件
- 自定义事件可以实现更灵活的开发，用好了有很多优势。与之相关的函数有 Event 构造函数，CustomEvent 和 dispatchEvent。
- 直接自定义事件，使用 Event 构造函数：

```javascript
var event = new Event('build') //new 一个Event 事件

div.addEventListener('build', function(event){}, false) //设置元素build监听事件

div.dispatchEvent('event') //触发事件
```

- CustomEvent 可以创建一个更高度自定义事件，还可以附带一些数据，具体用法如下：

```javascript
var myEvent = new CustomEvent(eventname, options)

var options = {
    detail: {					//detail 里存放一些初始化信息
        ...
    },
    bubbles: true,			//其他属性
    cancelable: false
}

div.dispatchEvent(myEvent) //手动触发事件

//结合起来使用即为：
obj.addEventListener('cat', function(event){}, false)
var event = new CustomEvent('cat', options) //自定义事件，options 为上述的那个

//使用 jQuery 磨平兼容性，调用方法如下
$('#div').on('cat', function(){}) //绑定事件
$('#div').trigger('cat') //触发事件
```

# 参考资料

[最详细的 js 事件](http://www.admin10000.com/document/6089.html) 

JS 高程