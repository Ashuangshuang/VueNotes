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







