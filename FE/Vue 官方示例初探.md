# Vue.js 官方示例初探
感谢作者 @尤小右 大大边写的超级带感的 Vue.js 前端框架，赠送的几个小例子都很有代表性，代码逻辑清晰简明，不禁想抄上一抄嗯。

官方的示例都是 ES5直接编写运行，并没有使用ES6以及构建工具，考虑到以后开发大一些的项目以及官方出品的 vue-cli脚手架，决定这次学习之旅采用两者结合写写官方的示例。

初探步骤：

1. 观摩示例的 result
2. 思考组件模板和逻辑实现思路
3. 遇到问题先搜一下 api 和官方教程（好像看过一遍还是记不住什么。。。结合实践重要嗯）
4. 还是不会就看例子的代码吧（不出意外的话都会走到这步哈哈）
5. 整理一下代码和总结

## markdown Editor
一个极简的 markdown 编辑器，用了 marked 这个工具。

- 新建一个 Marked 组件就 ok。看起来很简单，textarea 标签作为输入编辑器，另一个 div 标签通过 marked 这个 markdown工具 转码。
- npm i marked --save 来安装好 marked，import 后通过定义过滤器实现。
- textarea 设置了 debounce指令。debounce 设置一个最小的延时，在每次敲击之后延时同步输入框的值与数据。如果每次更新都要进行高耗操作（例如在输入提示中 Ajax 请求），它较为有用。

```javascript
<template>
  <div id="marked">
    <textarea v-model="input" debounce=500></textarea>
    <div v-html="input|marked"></div>
  </div>
</template>

<script>
  import marked from 'marked'
  export default {
    data () {
      return {
        // note: changing this line won't causes changes
        // with hot-reload because the reloaded component
        // preserves its current state and we are modifying
        // its initial state.
        input: '# Helzzz World!'
      }
    },
    filters: {
      marked: marked
    }
  }
</script>
```

## github commits

编写一个小组件，异步获取 github 的两条 branch的数据。

- created:生命周期的钩子，在实例创建后同步调用。此时实例已经结束解析选项，意味着已经建立了数据绑定，计算属性，方法，watcher/事件回调。但是还没有开始 DOM 编译，$el 还不存在。
- watch：一个对象，键是观察表达式，值是对应回调。值也可以是方法名，或者是对象，包含选项。在实例化时为每个键调用 $watch() 。
- **遇到的问题**：eslint 总是提示 new XMLHttpRequest() 错误，not defined，并不知道为啥会这样，看到了很多代码也并没出错啊，暂时在 eslint 的配置文件把 no-undef 设为0忽略它了，如果有知道的童鞋可以指点一二。

```javascript
<template>
  <div id="commits">
    <p>Latest vue.js Commits</p>
    <template v-for="branch in branches">
      <label :for="branch">
        <input type="radio"
               name="branch"
               :id="branch"
               :value="branch"
               v-model="currentBranch">
        {{branch}}</label>
    </template>
    <p>vuejs/vue@{{currentBranch}}</p>
    <ul>
      <li v-for="record in commits">
        <a :href="record.html_url" target="_blank">dd</a>
        - <span>{{record.commit.message}}</span>
        by <span>{{record.commit.author.name}}</span>
        at <span>{{record.commit.author.date}}</span>
      </li>
    </ul>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        branches: ['master', 'dev'],
        currentBranch: 'master',
        commits: null,
        apiURL: 'https://api.github.com/repos/vuejs/vue/commits?per_page=3&sha='
      }
    },
    created () { // 生命周期 created,获取数据
      this.fetchData()
    },
    watch: {  // 观测变化,可以是值也可以是方法
      currentBranch: 'fetchData'
    },
    methods: {
      fetchData () {
        let xhr = new XMLHttpRequest()
        const self = this  // 下面的 onload事件中 this 不再指向实例,所以要变量存一下
        xhr.open('GET', this.apiURL + this.currentBranch)
        xhr.onload = function () {
          self.commits = JSON.parse(xhr.responseText)
        }
        xhr.send()
      }
    }
  }
</script>
```

## Validation+Firebase
firebase 实时后端云简单搂了一眼，号称无后端数据存储加实时通信还是很带感的，不过自己写的时候总是报错，只好自己在本地 mock 一下了。以后写可以使用 wilddog，国内的

- 计算属性：由于模板中只可以用表达式，相对复杂的逻辑并不适合放在模板中，所以计算属性就派上用场了，简单易用。计算属性默认只是 getter 函数，不过也可以自定义 getter 和 setter函数。
- transition 过渡：这个过渡系统听勾股大大说很值得学习，所以暂时放下以后看源码先。
- mock 数据对象以后比较蛋疼，会把 newUser这个对象直接 push 进 userRef 中，导致以后对 newUser 的操作都会被双向绑定显示到列表中。。。所以只好深拷贝一下数据 push 进去，这个留坑以后填。
- !!:双叹号强制类型转换为布尔值。


```html
<template>
  <ul>
    <li class="user" v-for="user in userRef" transition>
      <span>{{user.name}} - {{user.email}}</span>
      <button @click="removeUser(user)">X</button>
    </li>
  </ul>
  <form @submit.prevent="addUser">
    <input type="text" v-model="newUser.name">
    <input type="text" v-model="newUser.email">
    <input type="submit" value="Add User">
  </form>
  <p v-show="!validation.name">Name can not be empty</p>
  <p v-show="!validation.email">email is not validated</p>
</template>
```

```javascript
<script type="text/babel">
  const emailRE = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/

  export default {
    data () {
      return {
        newUser: {
          name: '',
          email: ''
        },
        userRef: [
          {
            name: 'hh',
            email: '10218085@qq.com'
          },
          {
            name: 'heheihe',
            email: 'weirweoi@qq.com'
          }
        ]
      }
    },
    computed: {
      validation () {
        return {
          name: !!this.newUser.name.trim(),
          email: emailRE.test(this.newUser.email)
        }
      },
      isValid () {
        const validation = this.validation
        return Object.keys(validation).every(key => validation[key])
      }
    },
    methods: {
      addUser () {
        if (this.isValid) {
          let temp = JSON.parse(JSON.stringify(this.newUser))
          this.userRef.push(temp)
          this.newUser.name = ''
          this.newUser.email = ''
        }
      },
      removeUser (user) {
        
      }
    }
  }
</script>
```

## 树状视图
这个例子实现了树状视图，主要演示如何递归调用组件。

- 递归组件：组件在自身的模板内可以递归调用自己，但是**要有 name 选项才可以**，在这上面花了好长时间又去查了教程才发现。。。官方示例代码用 Vue.component()注册了全局组件，会把 id 自动注册为name 属性，所以没有手动写 name 属性。我在 cli 里写的时候一直没注意，导致递归总是不显示嗯。
- Vue.set:全局 API，设置对象的属性。如果对象是响应的，将触发视图更新。这个方法主要用于解决不能检测到属性添加的限制。
- `open = !open`:这是用来 toggle 布尔值，又学了一招~
- `@click和@dblclick`分别代表单击和双击事件绑定。后一个还真是没注意过。
- 动态 props：可以绑定 props，这样父组件数据变化后，也会传递给子组件。

```javascript
<template>
  <li>
    <div @click="toggle" @dblclick="changeType">
      {{model.name}}
      <span v-if="isFolder">[ {{open ? '-' : '+'}} ]</span>
    </div>
    <ul v-show="open">
      <Item :model="model" v-for="model in model.children">
      </Item>
      <li @click="addChild">+</li>
    </ul>
  </li>
</template>
```

```javascript
<script type="text/babel">
  import Vue from 'vue'
  export default {
    name: 'item',
    data () {
      return {
        open: false
      }
    },
    props: {
      model: Object
    },
    computed: {
      isFolder () {
        return this.model.children && this.model.children.length
      }
    },
    methods: {
      toggle () {
        if (this.isFolder) {
          this.open = !this.open
        }
      },
      changeType () {
        if (!this.isFolder) {
          Vue.set(this.model, 'children', [])
          this.addChild()
          this.open = true
        }
      },
      addChild () {
        this.model.children.push({name: 'new staff'})
      }

    }
  }
```

## 模态组件
一个弹出层，用到了组件、props、slot内容分发、过渡。

- slot:内容分发的一个东东，个人理解好像电脑主机的 pci 插槽，接口标准一致，可以插入不同的板子，比如 pci 网卡或者 pci声卡，根据 name 不同可以实现具名 slot，有需求的话别忘了设置 default的默认 slot嗯。
- Props:写 react 的时候就接触过了，父组件可以通过 props 向下传递数据，这里又踩了坑，**子组件一定要显式声明 props 属性！！！！**折腾了好久才去查 api 和教程，真是浪费时间。。

```javascript
<template>	// 父组件
  <div id="app">
    <button @click="showModal = true">show modal</button>
    <modal :show.sync="showModal">
      <h3 slot="header">heaaaaaader</h3>  具名 slot 传入
    </modal>
  </div>
</template>

<template> // 子组件
  <div v-show="show" class="modal-mask" transition="modal">
    <div class="modal-wrapper">
      <div class="modal-container" transition="modal">

        <div class="modal-header">
          <slot name="header">default header</slot>
        </div>

        <div class="modal-body">
          <slot name="body">default body {{show}}</slot>
        </div>

        <div class="modal-footer">
          <slot name="footer">default footer</slot>
          <button @click="show = false" class="modal-default-button">OK</button>
        </div>

      </div>
    </div>
  </div>
</template>
```

```javascript
<script type="text/babel">
  export default {
    props: {		//子组件一定别忘了声明 props
      show: Boolean
    }
  }
</script>
```

