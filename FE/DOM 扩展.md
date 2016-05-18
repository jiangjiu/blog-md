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
### getElementByClassName() 方法
- 可以通过 document对象和所有的 HTML 元素进行调用，原生的实现让他具有极大性能优势。
- 接受一个参数，一个包含一或多个类名的字符串,返回带有指定类的所有元素的 NodeList。传入类名时，顺序先后不重要。
- 


