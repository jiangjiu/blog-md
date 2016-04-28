# script 标签中的 defer和 async
## defer
- 可选属性。表示脚本可以延迟到文档被完全解析和显示后执行。只对外部脚本有效。

## async
- 可选属性。表示应该立即下载脚本，但不应该妨碍页面中的其他操作，比如下载其他资源或等待加载其他脚本。只对外部脚本有效。

### 例如

```javascript
<script async src="myAsyncScript.js"></script>  
<script defer src="myDeferScript.js"></script>
```

## 不存在 defer 和 async 属性时
- 通常情况下，无论如何包含代码，只要不存在 defer 和 async 属性，浏览器都会按`<script>`元素在页面出现的先后顺序对他们进行依次解析，当浏览器遇到的是外部 script 时，解析过程暂停，并发送请求下载 script 文件，当完全下载并执行后才会继续 DOM 解析。
- 如果不对 script 文件进行特殊处理，通常会阻塞页面。

## script 脚本带了 defer 属性
- 当页面有 N 个外链脚本放在 `<head>` 时，加载脚本会阻塞页面渲染，也就是常说的空白。简单的处理办法就是把 script 标签放在`</body>`前，但当开发环境越来越复杂时，不具有较强可维护性，可能需要反复调整。
- 当加入 defer 属性后，虽然 js 文件延迟加载，但是页面渲染不因 js 文件阻塞，DOMReady 的时间提前，明显感觉页面加载变快了。
- defer 属性会按照原本的 js 顺序执行，所以前后有依赖关系的 js 文件都可以放心使用。

## script 脚本带了 async 属性
- 当加入 async 属性后，js 下载时不会阻塞其他资源的加载，也不影响页面渲染。
- 需要注意的是，async 属性的 js 文件一旦下载好就会执行，所以很有可能并不按照上下位置的顺序执行，如果 js 前后具有依赖性，不推荐使用 async 属性。

## 总结
- defer 和 async 在网络加载时基本一致，都是异步加载。
- 他们的区别在于脚本下载完何时执行，defer 类似于脚本文件位于`</body>`前，显然更符合我们大部分的执行需求。 async 属性乱序执行，不管声明的顺序如何，只要下载完毕就会立即执行。
- 因为 async 完全不考虑依赖性，可以适用于 Google Analytics 等.
- 浏览器确保多个 defer 脚本按其在HTML页面中的出现顺序依次执行,且执行时机为DOM解析完成后，document的DOMContentLoaded 事件触发之前。

## 参考
[http://ued.ctrip.com/blog/script-defer-and-async.html](http://ued.ctrip.com/blog/script-defer-and-async.html)

[http://www.2cto.com/kf/201412/364116.html](http://www.2cto.com/kf/201412/364116.html)

[http://blog.csdn.net/renfufei/article/details/10210949](http://blog.csdn.net/renfufei/article/details/10210949)
