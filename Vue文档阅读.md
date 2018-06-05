Vue.js 的核心是一个允许采用简洁的模板语法来声明式地将数据渲染进 DOM 的系统。
# 文本插值
```
<div id="app">
  {{ message }} //文本插值：在模板中使用data中的数据不需要this,可以直接使用
</div>
var app = new Vue({
  el: '#app',//el声明挂载的element
  data: {
    message: 'Hello Vue!'
  }
})
```
app是Vue的实例，Vue会将data绑定到实例上，也就是可以直接通过app.message访问data中的message。
当然也可以通过app.$data.message访问。
Vue已经通过defineproperty对message设置了get，set方法，在get方法中定义watcher（观察者），无论通过哪种方法改变message都会触发watcher.notify方法，然后在watcher中更新视图，比如 this.node.nodeValue = this.value。v-bind的compiler将模板中{{}}替换成data中的数据。

# v-bind绑定元素特性
```
<div id="app-2">
  <span v-bind:title="message"> //同样直接访问data中的数据，不需要this
    鼠标悬停几秒钟查看此处动态绑定的提示信息！
  </span>
</div>

var app2 = new Vue({
  el: '#app-2',
  data: {
    message: '页面加载于 ' + new Date().toLocaleString()
  }
})
```
v-bind指令会在渲染的 DOM 上应用特殊的响应式行为。
在这里，该指令的意思是：“将这个元素节点的 title 特性和 Vue 实例的 message 属性保持一致”
```
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>
```
# v-if
```
<div id="app-3">
  <p v-if="seen">现在你看到我了</p>
</div>
var app3 = new Vue({
  el: '#app-3',
  data: {
    seen: true
  }
})
```
```
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>
```


# v-for

```
<div v-for="(value, key, index) in object">
  {{ index }}. {{ key }}: {{ value }}
</div>
```
还可以用of遍历
```
<div v-for="item of items"></div>
```

v-for 也可以取整数。在这种情况下，它将重复多次模板。
```
<div>
  <span v-for="n in 10">{{ n }} </span>
  // 1 2 3 4 5 6 7 8 9 10
</div>
```

v-for可以渲染template
```
<ul>
  <template v-for="item in items">
    <li>{{ item.msg }}</li>
    <li class="divider"></li>
  </template>
</ul>
```

## v-for v-if
当它们处于同一节点，v-for 的优先级比 v-if 更高，这意味着 v-if 将分别重复运行于每个 v-for 循环中。
```
<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo }}
</li>
```
上面的代码只传递了未完成的 todos。

# v-on
监听事件
```
<button v-on:click="reverseMessage">逆转消息</button>

// .prevent 修饰符告诉 v-on 指令对于触发的事件调用 event.preventDefault()
<form v-on:submit.prevent="onSubmit">...</form>

<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>
```

# v-model
```
<div id="app-6">
  <p>{{ message }}</p>
  <input v-model="message">
</div>
var app6 = new Vue({
  el: '#app-6',
  data: {
    message: 'Hello Vue!'
  }
})
```
v-model实现表单输入和应用状态之间的双向绑定。注意与v-bind区分，v-bind是实现元素特性与data的绑定，而v-model是实现表单输入和data之间的绑定。
前面提到compiler将模板中{{}}替换成data中的数据。但对v-model，compiler会绑定更新函数，当model中的数据被修改时触发更新函数，将修改同步到data的数据中，达到通知set的目的。v-bind没有绑定更新函数是因为只能通过修改data中的数据来更新视图，视图并不能更新data。

# 组件声明
```
<div id="app-7">
  <ol>
    <!--
      现在我们为每个 todo-item 提供 todo 对象
      todo 对象是变量，即其内容可以是动态的。
      我们也需要为每个组件提供一个“key”，稍后再
      作详细解释。
    -->
    <todo-item
      v-for="item in groceryList"
      v-bind:todo="item"
      v-bind:key="item.id">
    </todo-item>
  </ol>
</div>
// 定义一个组件
Vue.component('todo-item', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})

var app7 = new Vue({
  el: '#app-7',
  data: {
    groceryList: [
      { id: 0, text: '蔬菜' },
      { id: 1, text: '奶酪' },
      { id: 2, text: '随便其它什么人吃的东西' }
    ]
  }
})
```
# 数据与方法
```
// 我们的数据对象
var data = { a: 1 }

// 该对象被加入到一个 Vue 实例中
var vm = new Vue({
  data: data
})

// 获得这个实例上的属性
// 返回源数据中对应的字段，提问：为什么这里会输出true
// 因为双向绑定的原理也只是通过defineProperty定义set和get，同时对象是引用类型，也就是对这个对象defineProperty
// 也即是说vm实例上的data和全局对象是指向同一块堆地址
vm.a == data.a // => true

// 设置属性也会影响到原始数据
vm.a = 2
data.a // => 2

// ……反之亦然
data.a = 3
vm.a // => 3
```

## Object.freeze()
Object.freeze() 方法可以冻结一个对象，冻结指的是不能向这个对象添加新的属性，不能修改其已有属性的值，不能删除已有属性，以及不能修改该对象已有属性的可枚举性、可配置性、可写性。也就是说，这个对象永远是不可变的。这个方法返回传递的对象，而不是创建一个被冻结的副本。、
注意Object.freeze（）方法是浅冻结的，也就是只能冻结一层。如下：
```
obj1 = {
  internal: {}
};

Object.freeze(obj1);
obj1.internal.a = 'aValue';

obj1.internal.a // 'aValue'
```
深冻结需要递归
```
// 深冻结函数.
function deepFreeze(obj) {

  // 取回定义在obj上的属性名
  var propNames = Object.getOwnPropertyNames(obj);

  // 在冻结自身之前冻结属性
  propNames.forEach(function(name) {
    var prop = obj[name];

    // 如果prop是个对象,包括数组和对象，冻结它
    if (typeof prop == 'object' && prop !== null)
      deepFreeze(prop);
  });

  // 冻结自身(no-op if already frozen)
  return Object.freeze(obj);
}

obj2 = {
  internal: {}
};

deepFreeze(obj2);
obj2.internal.a = 'anotherValue';
obj2.internal.a; // undefined
```
使用这个方法将无法对对象使用defineProperty，然后就没有办法追踪数据的变化。
```
var obj = {
  foo: 'bar'
}

Object.freeze(obj)

new Vue({
  el: '#app',
  data: obj
})
<div id="app">
  <p>{{ foo }}</p>
  <!-- 这里的 `foo` 不会更新！ -->
  <button v-on:click="foo = 'baz'">Change it</button>
</div>
```
## 一些实例上内置的实例方法
除了数据属性，Vue 实例还暴露了一些有用的实例属性与方法。它们都有前缀 $，以便与用户定义的属性区分开来。
```
var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})

vm.$data === data // => true
vm.$el === document.getElementById('example') // => true

// $watch 是一个实例方法,观察vm.a,watch的handler就是function (newValue, oldValue)这个函数
// 但是注意如果你需要在handle中使用this，那就不能使用箭头函数来定义handler，因为箭头函数的handler绑定父级上下文，而不能指向vue的实例。所以最好不要使用箭头函数在这里。
vm.$watch('a', function (newValue, oldValue) {
  // 这个回调将在 `vm.a` 改变后调用
})
```

# 生命周期钩子
1.new Vue() 创建一个空的vue实例
2.init event & lifecycle 初始化事件和生命周期。（这一步具体在干嘛= =）
3.beforeCreate
4.使数据响应式，将初始化好的事件和响应式的属性注入到实例上
5.created 在这一步，实例已完成以下的配置：数据观测 (data observer)，属性和方法的运算，watch/event 事件回调。也就是创建了一个完整的Vue实例。
然而，挂载阶段还没开始，$el 属性目前不可见。
6.有没有el这个属性，如果有，那么就去寻找是否有template属性。如果没有，等待vm.$mount(el)手动添加el属性
7.如果有template就渲染这个模板，如果没有template属性就寻找el对应的外部HTML作为模板。找到后将模板编译成render函数。
理解下有没有template的差异：
```
new Vue({
  el: '#app',
  template: '<div id="app"><p>模板在templated参数中找到了哟~</p></div>'//有template属性
})
```
```
<div id="app"><p>模板是在外部HTML中找到的~</p></div>
创建对象实例：
new Vue({
  el: '#app' //没有template
})
```
然后将模板编译成render函数，这个过程很复杂，参考
[聊聊Vue的template编译.MarkDown](https://github.com/answershuto/learnVue/blob/master/docs/%E8%81%8A%E8%81%8AVue%E7%9A%84template%E7%BC%96%E8%AF%91.MarkDown)
总而言之，render函数最后会返回一个VNode节点，这个VNode节点相当于一个根元素，所以这步进行了模板到VNode的转换
8.beforeMount
9.create vm.$el and repalce "el" with it.将之前编译模板生成的虚拟DOM构造成真实的DOM，将这个真实的DOM，也即是vm.$el替换掉模板中的el，得到页面。
10.mounted
  10.5数据改变后，会重新编译模板,进行patch
11.beforeDestroy
12.移除watcher，childComponent,event listeners
13.destroyed

# 模板语法
## v-once
通过使用 v-once 指令，你也能执行一次性地插值，当数据改变时，插值处的内容不会更新。但请留心这会影响到该节点上的其它数据绑定.
只渲染元素和组件一次。随后的重新渲染，元素/组件及其所有的子节点将被视为静态内容并跳过。
```
<!-- 单个元素 -->
<span v-once>This will never change: {{msg}}</span>
<!-- 有子元素 -->
<div v-once>
  <h1>comment</h1>
  <p>{{msg}}</p>
</div>
<!-- 组件 -->
<my-component v-once :comment="msg"></my-component>
<!-- `v-for` 指令-->
<ul>
  <li v-for="i in list" v-once>{{i}}</li>
</ul>
```

## v-html
双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML，你需要使用 v-html 指令：
```
<p>Using mustaches: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```
你的站点上动态渲染的任意 HTML 可能会非常危险，因为它很容易导致 XSS 攻击。请只对可信内容使用 HTML 插值，绝不要对用户提供的内容使用插值

## 特性 -- 这一段是什么意思..
Mustache 语法不能作用在 HTML 特性上，遇到这种情况应该使用 v-bind 指令：

<div v-bind:id="dynamicId"></div>
在布尔特性的情况下，它们的存在即暗示为 true，v-bind 工作起来略有不同，在这个例子中：

<button v-bind:disabled="isButtonDisabled">Button</button>
如果 isButtonDisabled 的值是 null、undefined 或 false，则 disabled 特性甚至不会被包含在渲染出来的 <button> 元素中。

## JavaScript 表达式支持
Vue.js 都提供了完全的 JavaScript 表达式支持。
```
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

// 不只是文本插值，在元素的属性上也支持JS表达式
<div v-bind:id="'list-' + id"></div>
```
模板表达式都被放在沙盒中，只能访问全局变量的一个白名单，如 Math 和 Date 。你不应该在模板表达式中试图访问用户定义的全局变量。--不是很理解


# computed method watch
文档写得很好了，看文档吧

# class和style绑定
在vue中想要改变样式，要么通过绑定class，要么直接绑定样式。
## 绑定class
### 对象语法
```
<div v-bind:class="{ active: isActive }"></div>
```
或者直接传入一个计算属性的对象	
```
<div v-bind:class="classObject"></div>
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```
### 数组语法
我们可以把一个数组传给 v-bind:class，以应用一个 class 列表：
```
<div v-bind:class="[activeClass, errorClass]"></div>
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```
渲染为：
```
<div class="active text-danger"></div>
```
如果你也想根据条件切换列表中的 class，可以用三元表达式：
```
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
```
这样写将始终添加 errorClass，但是只有在 isActive 是 truthy[1] 时才添加 activeClass。

不过，当有多个条件 class 时这样写有些繁琐。所以在数组语法中也可以使用对象语法：
```
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```

## 绑定样式
### 对象属性
```
<div v-bind:style="styleObject"></div>
data: {
//其实是一个 JavaScript 对象,CSS 属性名可以用驼峰式 (camelCase) 或短横线分隔 (kebab-case，记得用单引号括起来) 来命名
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
```
### 数组属性
v-bind:style 的数组语法可以将多个样式对象应用到同一个元素上，注意数组中是样式对象，不是样式

<div v-bind:style="[baseStyles, overridingStyles]"></div>

当 v-bind:style 使用需要添加浏览器引擎前缀的 CSS 属性时，如 transform，Vue.js 会自动侦测并添加相应的前缀。	

### 多重值
从 2.3.0 起你可以为 style 绑定中的属性提供一个包含多个值的数组，常用于提供多个带前缀的值，例如：

<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
这样写只会渲染数组中最后一个被浏览器支持的值。在本例中，如果浏览器支持不带浏览器前缀的 flexbox，那么就只会渲染 display: flex。


# key
```
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address">
</template>
```
由于Vue会尽可能复用组件，上面的代码中切换 loginType 将不会清除用户已经输入的内容。因为两个模板使用了相同的元素，<input> 不会被替换掉——仅仅是替换了它的 placeholder。
但是如果给input添加key，那就意味着这是两个完全不一样的元素，所以会被替换掉，而不是复用
```
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username" key="username-input">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input">
</template>
```

# v-show
```
<h1 v-show="ok">Hello!</h1>

```
v-show 的元素始终会被渲染并保留在 DOM 中。v-show 只是简单地切换元素的 CSS 属性 display。
v-show 不支持 <template> 元素，也不支持 v-else。
	
## v-if v-show
v-if 是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。

v-if 也是惰性的：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

相比之下，v-show 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。

一般来说，v-if 有更高的切换开销，而 v-show 有更高的初始渲染开销。因此，如果需要非常频繁地切换，则使用 v-show 较好；如果在运行时条件很少改变，则使用 v-if 较好。

# key


# 数组更新检测
为了检测数组的变化，Vue重写了（貌似）数组的一些方法，这些方法包括：
```
push()
pop()
shift()
unshift()
splice()
sort()
reverse()
```
也就是你调用这些方法，Vue是可以检测到数组的变化.
但是下面这两种情况无法检测到：
当你利用索引直接设置一个项时，例如：vm.items[indexOfItem] = newValue
当你修改数组的长度时，例如：vm.items.length = newLength

解决方法：
1.直接替换数组，总是返回一个新数组
```
example1.items = example1.items.filter(function (item) {
  return item.message.match(/Foo/)
})
```
Vue 为了使得 DOM 元素得到最大范围的重用而实现了一些智能的、启发式的方法，所以用一个含有相同元素的数组去替换原来的数组是非常高效的操作。--Diff算法

2.使用Vue.set来改变数组
```
var vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})
vm.$set(vm.items, indexOfItem, newValue)
Vue.set(vm.items, indexOfItem, newValue)
```

3.使用splice方法
```
vm.items.splice(indexOfItem, 1, newValue)
vm.items.splice(newLength)
```

# 对象更新检测
Vue 不能检测对象属性的添加或删除
同样可以使用Vue.set(object, key, value)
```
var vm = new Vue({
  data: {
    userProfile: {
      name: 'Anika'
    }
  }
})
Vue.set(vm.userProfile, 'age', 27)
vm.$set(vm.userProfile, 'age', 27)
```

如果你需要添加多个属性,注意这个方法是返回一个新的对象
```
// 返回一个新的对象
vm.userProfile = Object.assign({}, vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
// 无效：
Object.assign(vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```
# is
有些 HTML 元素，诸如 <ul>、<ol>、<table> 和 <select>，对于哪些元素可以出现在其内部是有严格限制的。而有些元素，诸如 <li>、<tr> 和 <option>，只能出现在其它某些特定的元素内部。

这会导致我们使用这些有约束条件的元素时遇到一些问题。例如：
```
<table>
  <blog-post-row></blog-post-row>
</table>
```
这个自定义组件 <blog-post-row> 会被作为无效的内容提升到外部，并导致最终渲染结果出错。幸好这个特殊的 is 特性给了我们一个变通的办法：
```
<table>
  <tr is="blog-post-row"></tr>
</table>
```

# 组件注册
Vue.component('my-component-name', { /* ... */ })