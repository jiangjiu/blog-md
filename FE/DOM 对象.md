# DOM对象
## 节点层次
- 文档节点 document 是每个文档的根节点。文档节点只有一个子节点，即 `<html>` 元素,称之为文档元素。在 HTML 中，文档元素始终是`<html>`元素。
- HTML 元素通过元素节点表示，特性通过特性节点表示，文档类型通过文档类型节点表示，注释则通过注释节点表示。总共有12种节点类型，这些类型都继承于一个基类型。

## Node 类型
- 每个节点都有一个 nodeType 属性，用于表明节点类型。12个数值常量表示，任何节点类型必居其一。例如：

1. Node.ELEMENT_NODE（1）
2. Node.ATTRIBUTE_NODE（2）
3. Node.TEXT_NODE(3)

- 为了确保浏览器兼容，最好还是将 nodeType 属性与数字进行比较，因为 IE 没有公开 Node 类型的构造函数。
- 并不是所有节点类型都收到浏览器支持，最常用的还是元素和文本节点。
- 对于元素节点，nodeName 中保存的始终是元素的标签名，nodeValue 的值始终是 null。

### 节点关系
- 每个节点都有一个 childNodes 属性，保存着一个 NodeList 对象。这是一个类数组对象，就和 arguments 差不多。
- 它实际上是基于 DOM 结构动态执行查询的结果，因此 DOM 结构的变化能够自动反映到 NodeList 对象中，可以当做是**双向绑定**的对象。
- 和 arguments 对象一样，可以使用` Array.prototype.slice.call(someNode.childNodes)`将 childNodes 转换成数组进行操作，方括号访问和 item（）都是可以的，前者更常见些。
- IE8及以前需要手动枚举，所以上述代码会失效。
- 每个节点有一个 parentNode 属性，指向文档树中的父节点。通过previousSibling和 nextSbling 属性，可以访问其他节点。如果没有前一个或后一个，值为 null。

```javascript
if (someNode.nextSibling === null) {  //最后一个节点
	alert('Last node')
} else if (someNode.previousSibling === null) { //第一个节点
	alert('First node!')
}
```

- 如果只有一个节点，那么这两个属性都会为 null。
- 父节点的 firstChild 和 lastChild 属性分别指向其 childNodes 列表中的第一个和最后一个节点。`someNode.firstChild `始终等于` someNode.childNodes[0]`,`someNode.lastChild `始终等于` someNode.childNodes[someNode.childNodes.length-1]`。如果没有子节点，那么 firstChild 和 lastChild 始终为 null。
- 所有节点都有一个 ownerDocument 属性，指向整个文档的文档节点。

```javascript
nod.ownerDocument  //指向 document 节点
```

### 操作节点
- appendChild（）向 childNodes 列表末尾插入一个节点，默认返回新增节点。

```javascript
var returnNode = someNode.appendChild(someNode)
console.log(returnNode === someNode) //true  返回值即为插入的节点
console.log(someNode.lastChild === someNode) //TRUE 最后一个节点为新插入的节点
```

- 如果传入到 appendChild（）中的节点已经是文档中第一部分了，原位置就没有了这个节点，插入到新位置，相当于做了一次移动。

- insertBefore（）方法可以插入到某个特定位置，接受两个参数，插入的节点和作为参照的节点。

```javascript
returnNode = someNode.insertBefore(newNode, null)
alert (returnNode === someNode.lastChild) //true  插入成为最后一个节点

returnNode = someNode.insertBefore(newNode, someNode.firstChild)
alert (returnNode === someNode.firstChild) //true 插入成为第一个节点
```

- replaceChild（）方法接受两个参数，插入的节点和要替换的节点，替换。
- removeChild（）方法接受一个参数，移除节点。
- cloneNode（）方法克隆节点，接受一个布尔值，true 表示深复制，也就是节点和整个子节点树。false 表示只复制节点本身。**需要注意的是，这个节点返回的是复制后的节点，还需要手动方法把它插入到文档树中去。**
- normalize()方法处理文档树中的文本节点，如果找到空文本节点就删除它，如果找到相邻的文本节点就合并为一个文本节点。

## Document 类型
- nodeType的值为9，nodeValue 的值为'#document'
- document.domain可以设置。当页面包含来自其他子域的框架和内嵌框架时，能够设置 document.domain 就非常方便。如果两个页面的 document.domain 的值设置为一样，他们就可以进行通信了。如果域名一开始是松散的，那么就不可以在设置成紧绷的了。

```javascript
//假设页面来自于p2p.wrox.com
document.domain = 'wrox.com'  //成功，因为设置成了更松散的
document.domain = 'p2p.wrox.com' //失败！因为设置成了更紧绷的域名
```

- 查找元素的两个方法：document.getElementById（），document.getElementByTagName（）
- 第一个只返回第一次出现的元素，第二个返回的类数组对象HTMLCollection，也就是所有标签名一致的元素节点组，注意大小写尽量严格匹配，如果没找到会返回 null。
- document.getElementByTagName()方法返回的类数组对象可以用 nameItem（）访问也可以用方括号访问，传入 name 值。星号'*'代表全部。

## Element 类型
- nodeType 的值是1，nodeName 的值为元素标签名，nodeValue 值为 null。
- HTML 中所有标签都为大写，如果不确定是否为 HTML 或者 xml 最好进行标签名转换，tolowerCase（）方法。不会出错。推荐做法。
- 取得特性：getAttribute（）,setAttribute(), removeAttribute()方法。
- setAttribute() 方法接受两个参数，特性名和值。如果特性存在，就会替换掉，如果特性不存在，直接创建并设置。
- 使用 document.createElement() 方法可以创建新元素。传入元素的标签名。

## Text 类型
- nodeType 的值为3，nodeName 值为'#text'，nodeValue 值为节点所包含的文本。没有子节点。
- 通过 nodeValue 属性或者 data 属性修改文本。
- 通过 document.createTextNode()创建文本节点。

```javascript
var textNode = document.createTextNode('<strong>Hello</strong> World!')
```

- 一般情况下每个元素只有一个文本子节点，但如果我们自行删除或增加文本节点，会出现不是一个文本节点的情况。使用 normalize（）方法进行去除空文本或者合并节点的操作。

```javascript
element.normailze()  //element 元素的文本节点就会被规范化
```

- Text 类型提供了一个与 normaolize（）相反的方法：splitText（）,会按传入的数字分割 nodeValue 值，变成两个文本节点。原来的文本节点将包含开始到指定位置之前的内容，新文本节点将包含剩下的文本。

```javascript
var element = document.createElement('div')
element.className = 'message'

var textNode = document.createTextNode('Hello world!')
element.appendChild(textNode)

document.body.appendChild(element)

var newNode = element.firstChild.splitText(5) //方法返回的是后面的文本
alert(element.firstChild.nodeValue)  //'Hello'
alert(newNode.nodeValue)  //'world!'
```

## Comment 类型
- nodeType 的值是8，nodeName 值为'#comment'，nodeValue 是注释的内容，没有子节点。
- Comment 类型和 Text 继承自相同的基类，拥有除了splitText()之外的所有字符串操作方法。也可以通过 nodeValue 和 data 取得内容。注释节点可以通过父节点访问。
- 这个节点很少进行操作，因为对算法鲜有影响。

## DocumentFragment 类型
- DocumentFragment 在文档中没有对应的标记。nodeType 值为11，nodeName 值为'#document-fragment',nodeValue为 null。
- 创建文档片段可以使用 document.createDocumentFragment()方法，通常认为一次性的插入文档片段比多次浏览器渲染性能要高得多，所以如果有多次插入文档流的操作可以使用这个方式。

## Attr 类型
- nodeType 的值为2，nodeName 值为特性的名称，nodeValue 值为特性的值。
- 实际上，使用 getAttrbute()，setAtrribute(),removeAttribute()方法比操作节点更为方便。

