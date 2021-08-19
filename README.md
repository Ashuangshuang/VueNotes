# Vue Notes

## 模板语法
> 插值语法： {{}} 里面可以写JS表达式，自动读取data中所有属性，用于解析标签体内容
> 
>指令语法：v-... 用于解析标签（属性、内容、绑定事件），里面写JS表达式，v-bind:href 等同于 :href

## 数据绑定
> 单向v-bind：数据只能从data流向页面
> 
> 双向v-model：数据不只能从data流向页面，也能从页面流向data，只能用于表单项，默认收集的是value值

❗️❗️❗️v-model的三个修饰符
- lazy 失去焦点在收集数据
- number 输入字符串转为有效的数字
- trim 去掉首尾的空格
```html
<input type="number" v-model.number="formData.age">
```

## MVVM（Model-View-ViewModel）
> M模型-data中的数据，V视图-模板，VM视图模型-Vue的实例对象（data中所有的属性都会在vm上，vm的属性以及原型链上的属性，在模板中可直接使用）

## 数据代理
- 通过vm对象代理data中的属性操作
- 数据代理的好处，更加方便操作data数据，不代理的话可以通过vm._data访问
- 原理：
  - 通过Object.defineProperty()把data中的所有属性添加到vm上 
  - 为每个添加的属性都指定getter/setter
  - 在getter/setter内部操作data中的数据

## 事件修饰符
- `prevent` 阻止默认行为
- `stop` 阻止冒泡，给子级写
- `once` 事件只触发一次
- `capture` 使用事件捕获模式，由外向内，写在父元素上，冒泡是由内向外
- `self` 只有`e.target`是当前自身元素时才触发事件，给父级写也可以阻止冒泡，默认冒泡时父级取到的`e.target`是子级
- `passive` 事件的默认行为立即执行，无需等待回调函数执行完毕，例如@wheel鼠标滚轮事件，会先执行回调函数，函数执行完毕后，滚动条才往下移动，加上`passive`后就会立即滚动
```vue
<a href="http://www.baidu.com" @click.prevent="showAlert">百度一下</a>
```
## 计算属性computed
> 页面多个地方使用computed中的同一个属性值时，会缓存下来，不会多次调用get()方法
> 
> 使用方法返回值会被多次调用

```js
new Vue({
  data: {
    firstName: '王',
    lastName: '梦',
  },
  computed: {
    // 完整写法
    fullName: {
      // 读取时调用
      get() {
        return this.firstName + this.lastName;
      },
      // 修改时调用
      set(value) {
        const arr = value.split('-');
        this.firstName = arr[0];
        this.lastName = arr[1];
      },
    },
    // 简写，大多数情况都会只是读取不修改值
    fullName() { // 相当于get()方法
      return this.firstName + this.lastName;
    }
  }
});
```
## 侦听属性watch
> 当被监视的属性变化时，回调函数自动调用
> 
> 监视属性必须存在，才能监视
```js
const vm = new Vue({
  data: {
    isHot: false,
    numbers: {
      a: 1,
      b: 2,
    },
  },
  watch: {
    isHot: {
      immediate: true, // 初始化时让handler调用一下
      handler(newVal, oldVal) { // 当属性发生改变时调用
      },
    },
    
    // 简写，只需要handler配置项
    isHot(newVal, oldVal) {},
    
    // 监视多级结构中的某个属性变化
    'numbers.a': {
      handler(newVal, oldVal) {},
    },
    
    // 深度监视，监视多级结构中所有属性的变化
    // 默认只监视第一级数据结构，因为numbers存的是地址，需要用新对象替换掉
    numbers: {
      deep: true,
      handler(newVal, oldVal) {},
    }
  }
});

// 第二种写法
vm.$watch('isHot', {
  immediate: true,
  handler(newVal, oldVal) {},
})
```
## computed VS watch
- computed 能完成的 watch 都能完成
- watch 能完成的 computed 不一定能完成，例如 watch 可以异步操作
- 重要原则：
  - ❗️ 被Vue所管理的函数最好写成普通函数，这样this指向才是Vue的实例对象vm
  - ❗ 不被Vue所管理的函数（定时器的回调函数，Ajax的回调函数，Promise的回调）最好写成箭头函数，这样this指向才是vm
  
```js
new Vue({
  watch: {
    number() {
      // 定时器的回调函数不受Vue控制，是浏览器定时管理模块控制，到时间后是JS引擎调用
      // 写成普通函数，JS引擎调用，this指向window
      // 箭头函数内部没有自己的this，会向上查找到number普通函数的this，则指向vm
      setTimeout(() => {}, 1000)
    }
  }
});
```
## 绑定样式
```html
<body>
<!--字符串写法-->
<div cladd="basic" :class="normal"></div>
<!--数组写法-->
<div cladd="basic" :class="classArr"></div>
<!--对象写法-->
<div cladd="basic" :class="classObj"></div>

<div cladd="basic" :style="{fontSize: fSize + 'px'}"></div>
<div cladd="basic" :style="styleObj"></div>
<div cladd="basic" :style="styleArr"></div>
</body>
<script type="text/javascript">
  new Vue({
    data: {
      mood: 'normal', 
      classArr: ['normal', 'sad'], 
      classObj: {
        normal: true,  // 应用normal类名
        sad: false,
      }, 
      fSize: 40,
      styleObj: {
        fontSize: '40px',
      },
      styleArr: [{
        fontSize: '40px',
      },{
        backgroundColor: 'pink',
      }]
    }
  });
</script>
```
## 条件渲染
- v-show 适用于切换频率较高的场景，不显示的DOM通过display:none控制，不能写在template上
- v-if 适用于切换频率较低的场景，不显示的DOM直接被移除，可以写在template上，与v-else-if/v-else一起使用，但是结构不能被打断

## 列表渲染
- `v-for`指令可用于遍历数组、对象、字符串、指定次数
```html
<ul>
  <li v-for="(it, index) in list" :key="index"></li>
</ul>
```
### key的作用

> key是虚拟DOM的标识，当数据发生变化时，Vue会根据新数据生成新的虚拟DOM
> 
> 根据【新虚拟DOM】和【旧虚拟DOM】进行差异比较
> 
> 比较规则：
> - 在旧虚拟DOM中找到与新虚拟DOM相同的key
>   - 若虚拟DOM中的内容没变，直接使用之前的真实DOM
>   - 若虚拟DOM中的内容变了，则生成新的真实DOM，然后替换掉页面之前的真实DOM
> - 在旧虚拟DOM中没有找到与新虚拟DOM相同的key
>   - 创建新的真实DOM，随后渲染到页面

### 用`index`作为`key`可能引发的问题
- 对数据进行逆序添加、删除等破坏顺序的操作，会产生没有必要的真实DOM更新，界面效果没问题，但是效率低
- 结构中含有输入类DOM（input），会产生错误更新，界面显示有问题

## Vue监视数据的原理
> Vue 会监视data中所有层次的数据，主要通过`Object.defineProperty()`方法中的`set()`实现
> 
> data中的数据含有`get()`和`set()`就可以实现响应式
> 
> 先操作vm中的数据进而操作_data，_data中的set()更新DOM
> 
> 监测对象中的数据
> - 通过`setter`实现监视，并且要在new Vue的时候就要初始化
> - ❗data中没有进行初始化，后追加的属性没有get()和set()方法，Vue不做响应式处理
> - 后追加的属性可以通过`this.$set()`或者`Vue.set()`实现响应式
> 
> 监测数组中的数据
> - 通过包裹数组更新方法实现
> - 调用原生的数组方法对数据进行更新，重新解析模板，更新DOM
> - 修改数组使用以下方法才会实现响应式 push/pop/shift/unshift/splice/sort/reverse，也可以使用`this.$set()`
> - 通过数组索引修改数组对象中的属性可以实现响应式
> 
> ❗️`this.$set()`不能给 vm 或者根数据对象（data下的第一级数据）添加响应式属性
> 
> 数据劫持就是将自定义的data中的数据经过处理都添加了get()/set()方法，每次修改set()中都能监听到
```js
new Vue({
  data: {
    student: {
      name: 'Tom',
      age: 19,
      hobby: ['唱歌', '打球'],
      friends: [{
        name: 'Slina',
        age: 20,
      }]
    }
  },
  methods: {
    add() {
      this.student.sex = '男'; // 页面不更新
      this.$set(this.student, 'sex', '男'); // 添加未初始化的响应式数据
      this.hobby[0] = '跳舞'; // 页面不更新
      this.$set(this.student.hobby, 0, '跳舞'); //  更新
      this.friends[0].age = 21; // 更新
    }
  }
});
```
## 过滤器filters
> 对要显示的数据进行特定格式化后再显示，适用于简单逻辑处理，不改变原数据，产生新的数据
```html
<body>
<div>{{ time | formatDate }}</div>
<!--传参写法-->
<div>{{ time | formatDate('YYYY') }}</div>
<!--串联写法-->
<div>{{ time | formatDate('YYYY') | sliceStr }}</div>
</body>
<script type="text/javascript">
  // 全局过滤器
  Vue.filter('sliceStr', function (value){
    return 'hell',
  });
  new Vue({
    data: {
      time: '161903930022', 
    },
    // 局部过滤器
    filters: {
      formatDate(value, str) { // time为第一个参数
        return 'hello'
      }
    },
  });
</script>
```
## 内置指令
- `v-text`向所在的节点渲染文本内容，会替换节点中的所有内容，但是不会解析html标签，与`{{}}`类似
- `v-html` 向节点中渲染包含html结构的内容
  - ❗️❗️❗️有安全性问题
  - 在节点内容上写js代码获取本地cookie信息传参给跳转的url上，会导致xss攻击
  - 一定要在安全的内容上写v-html，永远不要用在用户提交的内容上
- `v-cloak` 没有值，本质是标签属性，Vue实例创建完毕接管容器后，会删除此属性
  - 使用css配合可以解决网速慢时页面展示出{{xxx}}问题
```css
/*所有含有v-cloak属性的标签*/
[v-cloak] { display: none }
```
- `v-once` 没有值，所在节点初次动态渲染后，就视为静态内容，以后数据改变不会引起所在结构的更新，可以用于优化性能
- `v-pre` 没有值，跳过所在节点编译过程，可用在没有使用指令语法、插值语法的节点，加快编译

## 自定义指令
> 主要操作DOM元素
> 
> ❗️指令中函数的`this`都是`window`，不是`vm`
```html
<body>
<div v-big="n"></div>
<input type="text" v-fbind:value="n">
<!--多个单词指令名-->
<div v-big-number="n"></div>
</body>
<script type="text/javascript">
  // 全局 别的Vue实例也可以使用
  Vue.directive('fbind', {
    bind(element, binding){
      element.value = binding.value;
      }, 
    inserted(element, binding){
      element.focus();
      }, 
    update(element, binding) {
      element.value = binding.value;
    }
  });
  new Vue({
    data: {
      n: 1, 
    },
    // 局部指令
    directives: {
      // 函数写法（默认就是bind和update时调用）
      // 指令与元素成功绑定时（第一次加载时会被调用）
      // 指令所在的模板被重新解析时会被调用（当有数据发生变化时）
      big(element, binding){
        element.innerText = binding.value * 10;
      },
      // 对象写法内包含钩子函数
      fbind: {
        // 指令与元素成功绑定时调用，注意此时元素并没有插入页面
        bind(element, binding){
          element.value = binding.value;
        },
        // 指令元素被插入到页面时调用
        inserted(element, binding){
          element.focus(); // 默认自动获取焦点
        },
        // 指令所在的模板被重新解析时调用
        update(element, binding) {
					element.value = binding.value;
				}
      },
			'big-number'(element, binding){
				element.innerText = binding.value * 10;
			},
    },
  });
</script>
```
## 生命周期函数
初始化
- `beforeCreate` 无法通过vm访问data和methods，*在数据监测、数据代理之前执行*
- `created` `this`可以访问`data`和`methods`，*在数据监测、数据代理之后执行，然后开始解析模板生成虚拟DOM*
- `beforeMount` 页面呈现的是未经Vue编译的DOM结构，所有对DOM的操作，最终都不生效，*在将内存中的虚拟DOM转换为真是的DOM插入页面之前执行*
- ❗️`mounted` 页面呈现的是经过Vue编译的DOM结构，对DOM操作均生效，一般在此进行开启定时器，发送网络请求等初始化操作，*完成模板解析并把初始的真实DOM放到页面后调用，只调用一次*

更新
- `beforeUpdate` 数据是新的，但是页面是旧的，页面和数据未保持同步，*新数据和旧数据进行比对，完成页面更新之前执行*
-  `updated` 数据、页面都是新的

销毁
- ❗`beforeDestroy` vm中的data、methods、指令都处于可用状态，但是一般不会在这里操作数据，即使操作数据也不会出发更新，一般在此阶段关闭定时器，解绑自定义事件等收尾工作
- `destroyed` 销毁完毕，销毁后自定义事件会失效，但是原生DOM事件依然有效

## 组件
> 实现应用中局部功能代码和资源的集合
> 
> 组件中定义的`data`必须使用函数式定义`data(){ return {  }}`，避免组件复用，数据存在引用关系
> 
> 如果使用对象定义，内存中存储的是同一个地址，有一个值改变，所有地方都会改变

### 非单文件组件
一个文件中包含n个组件
```js
// 创建组件使用Vue.extend()方法
// el 不要写，data必须写成函数式，使用template定义结构
// 简写 const school = {}; Vue会自动执行Vue.extend
const school = Vue.extend({
  template: `
    <div>
      <h2>{{name}}</h2>
      <h2>{{address}}</h2>
    </div>
  `,
  data() {
    return {
      name: '曼哈顿',
      address: '北京朝阳'
    }
  }
});

new Vue({
  // 局部注册
  components: {
    school
  }
});

// 全局注册
Vue.component('school', school);
```
### VueComponent
- `school`本身是一个名为`VueComponent`的构造函数，是`Vue.extent()`中返回的
- 在写`<school /> `时，Vue解析时会自动创建`school`组件的实例对象，即执行`new VueComponent(opts)`
- ❗在每次调用`Vue.extent()`时，返回的都是一个新创建的`VueComponent`
- 组件配置中的定义的函数`this`指向都是【`VueComponent`的实例对象，简称vc】，`new Vue(opts)`配置中的函数`this`指的是【`Vue`的实例对象，即vm】，vc与vm类似
- ❗定义组件时不可传入`el`参数，`el`只能在`new Vue(opts)`时传入

### ❗Vue与VueComponent的关系
- VueComponent的显示原型对象上的隐式原型指向Vue的显示原型对象
```js
VueComponent.prototype.__proto__ === Vue.prototype;
```
- 这样可以让组件实例对象`vc`可以访问`Vue`原型上的属性、方法

### 单文件组件
一个文件中包含一个组件

## Vue CLI
- 全局安装`npm install -g @vue/cli`后可使用`vue`命令
- 创建项目`vue create vue-test`
- `yarn serve` 启动项目
- 在`vue.config.js`中修改配置文件

## ref属性
- 应用在html标签上获取的是真是的DOM元素
- 应用在组件标签上获取的是组件的实例对象（vm）
- 通过`this.$refs.school`获取

## props
组件内部不能修改传入的`props`，如需修改将`props`放一份到`data`中，修改`data`
```js
export default {
  data() {
    return {
      myName: this.name,
    }
  },
  props: {
    name: {
      type: String,
      required: true,
      default: 'lala'
    }
  } 
}
```
## mixins混入
把多个组件公用的配置项提取成一个混入对象
- 生命周期函数如果混入对象、和组件内部都写了，那么两者都会执行，先执行混入对象中的钩子
- 值为对象的选项，例如 methods、components 和 directives，两者都写时，以组件中的为准
```js
const myMixin = {
  data() {
    return {
      name: 'lala'
    }
  },
  created () {
   console.log('myMixin-created');
  },
  methods: {
    hello() {
      console.log('hello from mixin!')
    }
  }
};

export default {
  // 局部混入
  minxins: [myMixin],
  data() {
    return {
      name: '赛林达',
    }
  },
}
// 全局混入
Vue.mixin(myMixin)
```
## 插件
> 用于增强Vue
> 
> 包含`install()`方法的一个对象，第一个参数是`Vue`，后面是使用者传递的参数
```js
// plugins.js 定义插件
export default {
  install(Vue, options) {
    // 添加全局过滤器
    Vue.filter();
    Vue.directive();
    // 添加实例方法
    Vue.prototype.$myMethod = function() {}
  }
}
// main.js
// 使用插件
Vue.use(plugins); // install()会自动调用
```
## style scoped
> 让样式在局部生效，防止冲突
> 
> 在style标签中写`scoped`属性表示当前样式只应用于当前组件
> 
> 编译后会在写样式的标签上生成一个属性，使用`demo[data-xxxxx]`作为新的类名
> 
> `npm view less-loader versions` 显示`less-loader`的版本记录
```html
<style scoped>
  .demo {
    background: orange;
  }
</style>
<!--需要安装less-loader-->
<style lang="less">
	.demo {
		background: orange;
	}
</style>
```
## 自定义事件
- 绑定
> 通过`this.$emit()`触发
> 
> 给组件写原生事件，组件会当成自定义事件，添加`@click.native`修饰符会默认绑定组件的最外层容器上
```html
<Student @getName="getName" />
<!--ref绑定 可以实现异步绑定-->
<Student ref="student" />
<script type="text/javascript">
  new Vue({
    methods: {
      getName(){
        
      }
    },
    mounted() {
      this.$refs.student.$on('getName', this.getName);
      
      // 注意：这种写法this指向是Student组件实例，回调函数写成箭头函数this指向Vue实例
      this.$refs.student.$on('getName', function() {
      });
      // this.$refs.student.$once 只绑定一次，只能调用一次
    }
  })
</script>
```
- 解绑
> 通过`this.$off()`解绑自定义事件
> 
> 传入数组解绑多个自定义事件
> 
> 不传参数解绑所有组件实例的自定义事件

## 全局事件总线（GlobalEventBus）
1. 一种组件间的通信方式，适用于任意组件间通信
2. 安装全局事件总线
```js
// main.js
new Vue({
  beforeCreate() {
    Vue.prototype.$bus = this; // this指向Vue的实例，就可以使用$on/$emit/$off方法了
  }
});
```
3. 使用事件总线
- 接收数据：A组件想要接收数据，则在A组件中给`$bus`绑定自定义事件，事件的回调留在A组件中
```js
export default {
  mounted() {
    this.$bus.$on('hello', this.demo)
  },
  methods: {
    demo(data) {
      // data为接收的数据
    }
  }
}
```
- 提供数据：`this.$bus.$emit('hello', 'data')`
4. 最好在`beforeDestroy`钩子中，通过`this.$bus.$off('hello')`解绑当前组件所用到的事件，因为组件销毁时绑定的自定义事件会销毁，但是`this.$bus`上绑定的事件不会自动销毁

## 消息订阅与发布（pubsub）
1. 一种组件间的通信方式，适用于任意组件间通信
2. 使用步骤
  - 安装pubsub`npm i pubsub-js`
  - 引入`import pubsub from 'pubsub-js'`
  - 接收数据
```js
export default {
  mounted() {
    // 订阅消息
    this.pubId = pubsub.subscibe('hello', this.demo)
  },
  methods: {
    demo(msgName, data) {
      // msgName：消息名 data为接收的数据
    }
  }
}
``` 
  - 提供数据：`pubsub.publish('hello', 'data')`
  - 最好在`beforeDestroy`钩子中，通过`pubsub.unsubscibe(this.pubId)`取消订阅

## this.$nextTick()
> 在执行方法时不会改变数据就更新视图，会在方法体全部执行完之后再更新视图
> 
> 在下一次DOM更新结束后再执行回调函数
> 
> 当改变数据后，要基于更新后的DOM进行某些操作时使用

## 过渡与动画
> 使用`transition`标签包裹元素，并配置`name`属性作为样式的标识
> 
> 通过配置进入和离开的固定类名语法和样式实现
> 
> 若多个元素需要过渡，需要使用`<transition-group>`，且每个元素都要写唯一`key`值
- 过渡写法
```vue
<template>
  <transition name="hello" appear>
    <h1>Hello</h1>
  </transition>
</template>
<style scoped>
h1 {
  background-color: orange;
}
/*进入的起点、离开的终点*/
.hello-enter, .hello-leave-to {
  transform: translateX(-100%);
}
/*进入的终点、离开的起点*/
.hello-enter-to, .hello-leave {
  transform: translateX(0);
}
.hello-enter-active, .hello-leave-active {
  transition:0.5s linear;
}
</style>
```
- 动画写法
```vue
<template>
  <transition name="hello" appear>
    <h1>Hello</h1>
  </transition>
</template>
<style scoped>
h1 {
  background-color: orange;
}
.hello-enter-active {
  animation: hello 0.5s linear;
}
.hello-leave-active {
  animation: hello 0.5s linear reverse;
}
@keyframes  hello{
  from {
    transform: translateX(-100%);
  }
  to {
    transform: translateX(0);
  }
}
</style>
```
## 插槽
> 让父组件可以向子组件指定位置插入html结构，通过作用域插槽子组件可以向父组件传入数据
- 默认插槽
```vue
<div>
  <slot>默认值</slot>
</div>
<!--使用-->
<Student>
  <p>学生姓名</p>
</Student>
```
- 具名插槽
> ❗️最新写法`v-solt:footer`只能应用在`template`或者组件上
```vue
<div>
  <slot name="center">默认值</slot>
  <slot name="footer">默认值</slot>
</div>
<!--使用-->
<Student>
  <p slot="center">斯黛拉</p>
  <p slot="footer">娜依扎</p>
  <p slot="footer">空白格</p>
  <template v-solt:footer></template>
</Student>
```
- 作用域插槽
> 必须在`template`上面写`scope="data"`或者新写法`slot-scope="{ games }"`才能接收到数据
> 
> 也可以起名字
```vue
<template>
  <div>
    <slot :games="games">默认值</slot>
  </div>
</template>
<!--使用-->
<Category>
  <template scope="data">
   <ul>
     <li v-for="it in data.games">{{it}}</li>
   </ul> 
  </template>
</Category>
```







