# JSON格式详解

JSON是一种数据格式，又是 JS 的一个严格的子集，但并不从属于JavaScript.

## 语法

1. 简单值：字符串、数值、布尔值、null.**不支持 js 中的特殊值 undefined。**
2. 对象：一种复杂数据类型，无序键值对儿。其中的值可以为简单值，也可以是复杂数据类型值。
3. 数组，一种复杂数据类型，有序的值的列表，可以通过数值索引访问，数组的值也可以是任意类型。

### 简单值

- js 字符串与 JSON 字符串最大的不同，在于 JSON 字符串必须为双引号，单引号可能导致语法错误。
- 布尔值和 null 也是有效的 JSON 格式，但通常是会用复杂的数据结构表示 JSON 对象。

### 对象

- js 对象字面量和 JSON 格式大体相同，但 JSON 对象有两点不同，首先没有变量声明，也没有末尾的分号，另外值得注意的是，**属性必须加双引号**！忘了给属性加双引号，或者写成单引号都是常见错误。

```javascript
{
	"name": "Nicholas",
	"age": 29,
	"school": {
		"name": "BUPT",
		"location": "beijing"
	}
}
```

## 数组

- JSON 数组也没有变量和分号，对象和数组通常是 JSON 数据结构的最外层形式。但并不强制。

## 解析和序列化

JSON之所以流行，主要的是可以把 JSON 数据结构解析为有用的 js 对象。简单清晰明了。

### JSON.stringify()

#### JSON.stringify()方法序列化的内部顺序：

1. 如果存在 toJSON()方法，首先调用该方法，否则返回对象本身。
2. 如果提供第二个参数，应用这个函数过滤器，第一个参数值为第一步传入的值。
3. 对第二步返回的每个值进行相应的序列化。
4. 如果提供了第三个参数，执行相应的序列化。


- JSON.stringify()用于把 js 对象序列化为 JSON 字符串。

```javascript
var book = {
	title: 'professional js',
	authors: [
		'Nicholas'
	],
	edition: 3,
	year: 2001
}

var jsonText = JSON.stringify(book)  //jsonText 即为转换好的字符串

//{"title":"professional js","authors":["Nicholas"],"edition":3,"year":2001}
console.log(jsonText)
```

- JSON.stringify()其实还可以接受另外两个参数，用于以不同的方式进行序列化。
- 第一个参数是个过滤器，可以是数组也可以是函数，第二个参数表示是否保留缩进。这两个参数可以单独或配合使用。
- 如果第一个传入的参数是数组，那么返回的结果只会包含数组列出的属性。

```javascript
//得到的JSON 字符串只有 title 和 edition 两个属性
var jsonText = JSON.stringify(book, ["title", "edition"]) 
```

- 如果第一个传入参数是函数，函数接受(key, value)两个参数，根据属性名分别处理属性，属性名只能是字符串。如果返回值是 undefined，那么相应的属性就不会出现在返回的 JSON 字符串中了。相当于删除该属性。

```javascript
var jsonText = JSON.stringify(book, function(key, value){  //传入一个函数处理 key value
	switch(key){
		case "authors":				
			return value.join(',')	//没有 break 注意
			
		case 'year':
			return 5000
			
		case 'edition':
			return undefined  //相当于删掉这个属性
			
		default:
		return value
	}
})
```

- 第三个参数用于处理缩进和空白符。如果是数值，那么代表缩进的空格数。如果是字符串，代表空白符。
- 无论是数值还是字符串，都只会保留10位，超出的会被忽略，结果中只会出现10个空格或前十个字符。

```javascript
var jsonText = JSON.stringify(book, null, 4) //缩进4个字符
var jsonText = JSON.stringify(book, null, '----') //缩进4个制表符
```

- toJSON()方法。任何对象都可以添加 toJSON 方法，通过设置返回值使 JSON.stringify()生效。如果返回值为 undefined，如果是顶级对象，结果是 undefined，如果是包含在其他对象中，它的值为 null。


```javascript
 var book = {
    title: 'professional js',
    authors: [
      'Nicholas'
    ],
    edition: 3,
    year: 2001,
    toJSON: function () {  //自定义 toJSON()方法
      return this.title
    }
  }

  var jsonText = JSON.stringify(book)
  console.log(jsonText)  // 返回的JSON 对象为"professional js"
```

### JSON.parse()

- JSON.parse()用于把 JSON 字符串解析为原生 js 值。
- 和 JSON.stringify()方法类似，parse()也可以接受第二个参数，是一个函数，将在每一个键值对上调用。参数也是 key、value 的形式。

```javascript
var bookCopy = JSON.parse(jsonText, function(key, value) {
	if (key === "releaseDate") {	//通过判断 key，来进行相应 value 的操作
		return new Date(value)
	} else {
		return value
	}
})
```