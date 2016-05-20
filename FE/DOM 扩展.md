# DOM 扩展
- 对 DOM 的两个主要的扩展是 Selectors API 和 HTML5.

## 选择符 API
- Selectors API Level 1的核心是两个方法: querySelector()和 querySelectorAll().兼容性 IE8+。

### querySelector方法
- querySelector()方法接受一个 CSS 选择符，返回与该模式匹配的第一个元素，没有匹配返回 null。

```javascript
var body = document.querySelector('body') //获取 body 元素
var myDiv = document.querySelector('#myDiv') //获取 ID 为 myDiv 的 div 元素
var selected = document.querySelector('.selected') //获取 class 为 selected 的第一个元素
var img = document.body.querySelector('img.button') //获取类为 button 的 第一个 img 元素
```

- 通过 Document 类型调用 querySelector()方法时，会在文档元素范围内查找，通过 Element 类型查找时，则在该元素的后代元素范围内查找。
- 如果传入了不被支持的选择符，会报错。

### querySelectorAll方法
-	同上，返回的是一个 NodeList 实例。
- 具体来说，返回值实际上是一个带有所有属性和方法的 nodeList，底层实现列斯雨一组元素的快照，并非对文档进行搜索的动态查询。这样实现可以避免使用 NodeList 对象引起的大多数性能问题。
- 传入的 CSS 选择符有效，则返回一个 NodeList 对象，可能为空，如果传入了错误的选择符则抛出错误。

## 元素遍历
- 对于元素间的空格，IE9以及之前的版本不会返回文本节点，其他浏览器会返回文本节点。导致了使用 childNodes 和 firstChild 等属性时的行为不一致。为了弥补这一不一致，保持 DOM 规范不变，Element Travelsal规范重新定义了一组属性。

API|说明
:---:|:---:
childElementCount | 返回子元素的个数
firstElementChild | 指向第一个子元素,firstChild 的元素版
lastElementChild | 指向最后一个子元素，lastChild 的元素版
previousElementSibling | 前一个同级元素，previousSibling 的元素版
nextElementSibling | 后一个同级元素，nextSiblingde 的元素版

- 利用这些元素不必担心空白文本节点，从而方便的查找元素。

```javascript
var child = element.firstElementChild   //元素的第一个子元素
while (child != element.lastElementChild) {  //并非最后一个子元素时，处理后，child 指向下一个子元素直至成为最后一个
	processChild(child)
	child = child.nextElementSibling
}
```

# HTML5
## 与类相关的扩充
### getElementsByClassName() 方法
- 可以通过 doucument对象和所有的 HTML 元素进行调用，原生的实现让他具有极大性能优势。
- 接受一个参数，一个包含一或多个类名的字符串,返回带有指定类的所有元素的 NodeList。传入类名时，顺序先后不重要。

```javascript
//取得所有类名为 username current 的元素，类名顺序无所谓
var allCurrentUsername = document.getElementSByClassName('username current')

//取得 ID 为 myDiv 的元素中带有类名selected 的所有元素
var selected = document.getElementById('myDiv').getElementsByClassName('selected')
```

- 和其他的选择器方法一样，只有位于调用元素树中的元素才会返回。
- 因为返回的对象是 NodeLst，所以使用这个方法与使用getElementsByTagName()以及其他返回 NodeList 的 DOM 方法都有相同的性能问题。
- **这里要多说两句，因为在循环中经常使用 i < selected.length 这样的语句，但每一次判断 i 是否小于类数组对象的长度时，都会对 NodeList 对象进行一次访问查询，导致性能下降。如果通过一个变量缓存这个 length，性能可以提升50%左右。所以尽量减少访问属性的次数。**

### classList 属性
- 所有元素都添加了了 classList 属性，是新集合类型 DOMTokenList 的实例。
- classList 对象有一个表示自己长度的 length 属性，要去的每个元素可以使用 item()方法或者方括号语法。

方法 | 说明
--- |:---:
add(value) | 将给定的字符串添加到列表中，如果已经存在，就不添加
contains(value) | 表示列表中是否存在给定的值，如果存在就不添加
remove(value) | 从列表中删除字符串
toggle(value) | 类似 jQuery 中的切换类名

- 有了 classList 属性，除非需要全部删除所有类名，或者完全重写元素属性，否则也用不到 className 属性了。

### 焦点管理
- document.activeElement 属性,这个属性会引用 DOM 当前获得焦点的元素，获得焦点的方式有页面加载，用户输入，focus()方法。

```javascript
var button = document.getElementById('button')
button.focus()
console.log(document.activeElement === button) // true
```

- 默认情况下，文档刚刚加载完成时，activeElement 属性保存的是 document.body 元素的引用。加载期间值为 null。
- 另外，document.hasFocus()方法确定文档是否获得了焦点。只要文档加载完成或者其他元素调用了 focus()方法，这个返回值为 true.

### 自定义属性
- HTML5规定可以为元素加载非标准的属性，添加前缀 data-，目的是为元素提供与渲染无关的信息，或者提供语义信息。这些属性可以任意添加，随便命名，只要以 data-开头即可。

```javascript
<div id='myDiv' data-appId='12345' data-myname='Nicholas'></div>
```

- 自定义属性可以通过元素的 dataset 属性访问。
- dataset 属性是 DOMStringMap 的一个实例，也就是一个键值对的映射。访问的属性名没有 data-前缀。

```javascript
var div = document.getElementById('myDiv')

var appId = div.dataset.appId
var myName = div.dataset.myname

div.dataset.appId = 23456
div.dataset.myname = 'Michael'
```

- 如果给元素添加一些不可见的数据一遍进行其他处理，在跟踪连接和混搭应用中，自定义属性能方便的知道点击来自哪里。

### 插入标记
- 通过 DOM 操作节点相对麻烦，因为不仅要创建，还要按照正确的顺序连接。使用插入标记的技术，直接插入 HTML 字符串更简单，速度也更快。

#### innerHTML 属性
- 读模式下，innerHTML 属性返回与调用元素的所有子节点（元素，注释，文本节点）对应的 HTML 标记。
- 写模式下，innerHTML 会根据指定的值创建新的 DOM 树，新的 DOM 树会完全取代原来的 DOM 节点。

```javascript

<div id="content">
  <p>this is a <strong>paragraph</strong>with a list following it</p>
  <ul>
    <li>Item1</li>
    <li>Item2</li>
    <li>Item3</li>
  </ul>
</div>

//对于上面的div 元素来说，它的 innerHTML 会返回如下字符串
  <p>this is a <strong>paragraph</strong>with a list following it</p>
  <ul>
    <li>Item1</li>
    <li>Item2</li>
    <li>Item3</li>
  </ul>
```

- 尽量注意，在通过 innerHTML 插入代码时，尽量收工检查一下其中的文本内容。

#### outerHTML 属性
- 基本同上，需要注意的是返回也会包括自身，看下面的例子。

```javascript

<div id="content">
  <p>this is a <strong>paragraph</strong>with a list following it</p>
  <ul>
    <li>Item1</li>
    <li>Item2</li>
    <li>Item3</li>
  </ul>
</div>

//对于上面的div 元素来说，它的 outerHTML 会返回如下字符串
<div id='content'>
  <p>this is a <strong>paragraph</strong>with a list following it</p>
  <ul>
    <li>Item1</li>
    <li>Item2</li>
    <li>Item3</li>
  </ul>
</div>  
```

#### insertAdjacentHTML()方法
- 接受两个参数，插入位置和要插入的 HTML 文本。
- 太乱，不写了。

```javascript
//作为一个同辈元素插入
element.insertAdjacentHTML('beforebegin', '<p>helloworld</p>')
```

#### 内存与性能问题
- 在使用 innerHTML、outerHTML、insertAdjacentHTML()方法最好收工删除要被替换的元素的所有事件处理程序和 Javascript 对象属性。因为在使用前述某个属性将该元素从文档树中删除后，元素与事件处理程序之间的绑定关系在内存中并没有一并删除。如果这种情况频繁出现，页面占用的内存数量就会明显增加。
- 再插入大量新 HTML 标记时，使用 innerHTML 属性与通过多次 DOM 操作先创建节点在指定他们的关系相比，效率高得多。因为设置 innerHTML 时，会创建 HTML 解析器（通常是 C++编写），比执行 js 快得多。
- 创建和销毁 HTML 解析器也会带来性能损失，所以最好不要频繁设置 innerHTML 等属性。

#### scrollIntoView()方法
- scrollIntoView()可以在所有 HTML 元素上调用，给方法传入 true 或者不传参，调用元素的顶部会与视窗顶部尽量平齐，如果传入 false，调用元素会尽量全部出现在视窗中。

```javascript
var div = document.querySelector('#b_fav')

div.scrollIntoView() //滚动至div 出现在视窗页面顶部
```

