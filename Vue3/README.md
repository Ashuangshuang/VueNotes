# Vue3 
- 2020-9-18发布3.0版本，代号One Piece（海贼王）

## 带来了什么
1. 性能提升
  - 打包大小减小41%
  - 初次渲染快55%，更新渲染快133%
  - 内存减少54%
2. 源码升级
  - 使用Proxy代替defineProperty实现响应式
  - 重写虚拟DOM的实现和Tree-shaking（去掉不用代码）
3. 更好的支持TypeScript
4. 新特性
  - 组合式API
  - 新的内置组件

## 创建Vue3工程
1.使用vue-cli
2.使用vite（下一代前端构建工具，尤雨溪团队打造）

## 常用Composition API（组合API）

### setup配置函数

- 组件所用到的数据，方法均要配置在setup中
- 有两种返回值
  - ❗️返回一个对象，对象中的属性在模板中均可直接使用
  - 返回一个渲染函数，可以自定义渲染内容，会替换掉页面的模板中内容
- 注意
  - 不要与vue2配置混用
  - vue2配置中可以访问setup中属性、方法，但在setup中不能访问2的配置
  - 如果有重名，setup优先
  - setup不能是一个async函数，因为async函数返回不在返回一个对象，而是promise，模板看不到return 中的属性
  - ❗️❗❗️但是️，后期也可以返回一个 Promise 实例，但是需要 `suspense` 和 `defineAsyncComponent`的配合
- 执行时机在`beforeCreate`之前执行一次，this是`undeined`
- 参数
  - props：值为对象，包含组件外部传递过来的，且组件内部声明接收了的属性
  - context：上下文对象
    - attrs 值为对象，包含组件外部传进来的，但是没有在`props`配置中声明，相当于`this.$attrs`
    - slots：收到插槽内容，相当于`this.$slots`
    - emit： 触发自定义事件，相当于`this.$emit`
```js
import { h } from 'vue'
export default {
  props: ['title', 'school'],
  setup(props, context) {
    let name = '李成阳';
    function sayHelllo() {
      alert(name)
    }
    return {
      name,
    }
    // 函数写法
    // return () => h('div', name)
  }
}
```

### ref函数
- 定义一个响应式数据的引用对象（reference对象，简称ref对象）
- 接收的数据可以是基本类型，也可以是对象类型
  - 基本类型数据响应式依旧靠`Object.defineProperty()`的`get`和`set`完成的
  - 对像类型的数据内部使用了新函数`reactive`，`reactive`函数内部写了具体对`Proxy`的实现
```vue
<template>
  <h1>{{name}}</h1>
  <h1>{{job.type}}</h1>
</template> 
<script>
import { ref } from 'vue'
export default {
  setup() {
    let name = ref('李成阳'); // 返回的是RefImple的实例对象
    let job = ref({ // 返回的也是ref对象，但是job.value是一个Proxy{ type: "..." }
      type: '前端',
      salary: '20k'
    }); // 对象
    function changeInfo() {
      name.value = '大江'; // 操作数据需要修改value值
      job.value.type = 'node js'; // type后不需要加value
    }
    return {
      name,
      changeInfo,
      job
    }
  }
}
</script>
```
### reactive函数
- 定义一个<font color="red">对象类型</font>的响应式数据，基本类型要用`ref`函数
```vue
<template>
  <h1>{{name}}</h1>
  <h1>{{job.type}}</h1>
</template> 
<script>
import { reactive } from 'vue'
export default {
  setup() {
    let job = reactive({ // 返回的直接是Proxy对象
      type: '前端',
      salary: '20k'
    });
    let hobby = reactive(['台球', '乒乓球']);
    function changeInfo() {
      job.type = 'node js';
      hobby[0] = '篮球'; // Proxy对象数组索引值修改也可以实现响应式
    }
    return {
      name,
      changeInfo,
      job
    }
  }
}
</script>
```
### reactive 对比 ref
- 定义数据
  - ref 基本数据类型
  - reactive 对象、数组类型
  - ref也可以定义对象、数组类型，内部会自动通过`reactive`转为代理对象
- 原理
  - ref 使用`Object.defineProperty()`的`get()`和`set()`实现响应式（数据劫持）
  - reactive 使用 Proxy 实现响应式（数据劫持），并通过 Reflect 操作源对象内部的数据
- 使用
  - ref 定义的数据操作需要使用`.value`，读取数据模板中直接读取
  - reactive 定义的数据均不需要`.value`

### Vue2响应式原理
- 对象类型：通过`Object.defineProperty()`对属性的读取、修改进行拦截（数据劫持）
- 数组类型：通过重写更新数组的一系列方法来实现拦截（对数组方法进行了包裹）
- 存在的问题
  - 新增属性、删除属性、界面不更新（需要单独通过`this.$set()`进行单独处理）
    - 因为对`Object.defineProperty()`对对象中属性的添加、删除捕获不到
  - 通过索引修改数组中的基本类型数据，界面不更新
  
### Vue3响应式原理
- 通过Proxy代理：拦截对象中任意属性的变化，包括读写、添加、删除
- 通过Reflect反射：对被代理对象的属性进行操作
```js
// 源数据（被代理对象）
let person = {
  name: '张三',
  age: 18
};

// 使用Proxy不用像Object.defineProperty遍历为每个属性添加getter/setter
// Reflect 反射对象，类似于Object
// 但是Reflect.defineProperty有返回值（返回true/false），定义相同的属性名时，代码会继续执行不会中断（相对友好一些），结果以先定义的为准
// Object.defineProperty定义相同的属性名时会报错，后面所有代码不执行
const p = new Proxy(person, {
  // 读取属性调用
  get(target, propName) {
    return Reflect.get(target, propName);
  },
  // 修改、追加属性调用
  set(target, propName, value) {
    Reflect.set(target, propName, value)
  },
  // 删除属性调用
  deleteProperty(target, propName) {
    return Reflect.deleteProperty(target, propName); // 返回true/false
  }
});
```
### computed函数
```vue
<script>
import { computed, reactive } from 'vue';
export default {
  setup() {
    let p = reactive({
      firstName: '郭',
      lastName: '仙女',
    });
    
    // 简写
    let fullName = computed(() => `${p.firstName}-${p.lastName}`);
    // 完整
    
    let fullName = computed({
      get() {
        return `${p.firstName}-${p.lastName}`;
      },
      set(value) {
        ...
      }
    });
    return {
      fullName
    }
  }
}
</script>
```
### watch函数
- <font color="red">坑</font>
  - 监视reactive定义的响应式数据的全部属性时，oldValue无法正确获取、强制开启了深度监视（deep配置失效）
  - 监视reactive定义的响应式数据中的某个对象属性时，deep配置有效，oldValue无法正确获取
- ❗️监视ref数据的基本类型时不需要`.value`，监视对象类型的数据时需要`.value`，也不能正确获取oldValue
  - 因为Ref对象中的value存的是Proxy对象的地址，直接监视Ref对象默认是浅层次的监视，想要深度监视有2种办法
  - 1. 监视时使用`.value`
  - 2. 监视时不使用`.value`，在后面配置`{ deep: true }`
  
```vue
<script>
import { ref, watch, reactive } from 'vue';
export default {
  setup() {
    let num = ref(1);
    let msg = ref('你好');
    let p = reactive({
      name: '郭',
      age: 18,
      job: {
        salary: 20,
        job2: {
          salary: 18
        }
      }
    });
    
    // 监听ref定义的响应式数据
    watch(num, (newV, oldV) => { }, { immediate: true })
    
    // 监听多个ref定义的响应式数据
    watch([num, msg], (newV, oldV) => { });
    
    // 监听reactive定义的响应式数据全部属性
    // 获取不到oldV，强制开启了deep: true，配置成false也无效
    watch(p, (newV, oldV) => {});
    
    // 监听reactive定义的响应式数据的某个属性
    // 注意第一个参数是函数
    watch(() => p.name, (newV, oldV) => {});

    // 监听reactive定义的响应式数据的多个属性
    watch([() => p.name, () => p.age], (newV, oldV) => {});

    // 监听reactive定义的响应式数据的值为对象的属性
    // 注意：此时deep:true有效，但是获取不到oldV
    watch(() => p.job, (newV, oldV) => {}, { deep: true });
    
  }
}
</script>
```
### watchEffect函数
- watch 既要指明监视的属性，也要指明监视的回调
- watchEffect 不用指明监视哪些属性，监视的回调中用了哪个属性，那就监视哪个属性
- watchEffect 与 computed类似
  - computed 注重计算出的结果，必须写返回值
  - watchEffect 注重过程（回调函数的函数体），不需要返回值
```vue
<script>
import { ref, watchEffect, reactive } from 'vue';
export default {
  setup() {
    let num = ref(1);
    let msg = ref('你好');
    let p = reactive({
      name: '郭',
      age: 18,
      job: {
        salary: 20,
        job2: {
          salary: 18
        }
      }
    });
    
    // 监听ref定义的响应式数据
    watchEffect(() => {
      const x1 = num.value;
      console.log('watchEffect------'); // num变化了就会执行
    })
  }
}
</script>
```
### 自定义hook函数
- 本质是一个函数，把setup函数中使用的Composition API进行了封装
- 类似于Vue2中的mixin
- 优势，可以复用代码，让setup中的逻辑更清晰

### toRef
- 作用：创建一个ref对象，其value值指向另一个对象（定义的对象）中的某个属性
- 应用：将响应式中的某个属性单独提供给外部使用
- 扩展：`toRefs`与`toRef`功能一致，但是可以批量创建多个ref对象
```vue
<template>
  <h1>{{name}}</h1>
  <h1>{{age}}</h1>
  <h1>{{job.job2.salary}}</h1>
</template>
<script>
import { toRefs, toRef, reactive } from 'vue';
export default {
  setup() {
    let p = reactive({
      name: '郭',
      age: 18,
      job: {
        salary: 20,
        job2: {
          salary: 18
        }
      }
    });
    // 注意toRef返回的ref对象中的value指向的是p中的数据
    // 如果使用ref则是新生成的ref对象，页面改变也是新生成的数据，并不是p中的
    return {
      name: toRef(p, 'name'),
      age: toRef(p, 'age'),
      job: toRef(p, 'job'),
    }
    
    // toRefs写法
    const data = toRefs(p); // 返回的是对象，对象的每个值时ref对象
    return {
      ...data
    }
    
  }
}
</script>
```
## 其他Composition API
### shallowRef 和 shallowReactive
- shallowReactive 只处理对象最外层属性的响应式（浅响应式）
- shallowRef 只处理基本类型数据的响应式，不进行对象响应式处理
- 什么时候使用
  - 对象数据结构比较深，但是只是外层属性变化，使用 shallowReactive
  - 对象数据后续功能不会修改对象中的属性，而是生成新的对象来替换，使用 shallowRef
  
### readonly 与 shallowReadonly
- readonly 让一个响应式数据变为只读的（深只读）
- shallowReadonly 让一个响应式数据变为只读的（浅只读）
- 应用场景：不希望数据被修改时

### toRaw 与 markRaw
- toRaw
  - 将一个`reactive`生成的响应式对象转换为普通对象
  - 使用场景：用于读取响应式对象对应的普通对象，对这个普通对象的所有操作，不会引起页面更新
- markRaw
  - 标记一个对象，使其永远不会在成为响应式对象
  - 使用场景
    - 有些值不应该设成响应式，例如复杂的第三方库
    - 当渲染不可变的数据源大列表时，跳过响应式转换可以提高性能
```vue
<script>
import { reactive, markRaw, toRaw } from 'vue';
export default {
  setup() {
    let p = reactive({
      name: 'Kitty',
      age: 18,
      job: {
        salary: 20,
        job2: {
          salary: 18
        }
      }
    });
    function addCar() {
      let car = { name: '奔驰' };
      p.car = car; // 响应式数据
      p.car = markRaw(car); // car不是响应式数据
      const raw = toRaw(p); // 将Proxy对象转换为原始对象，失去响应式
    }
  }
}
</script>
```
### customRef 自定义ref
创建一个自定义ref，对其依赖项追踪和更新出发进行显示控制
```js
import { customRef } from "vue";
function useDebouncedRef(value, delay = 200) {
  let timeout;
  // 通过customRef实现自定义
  return customRef((track, trigger) => {
    return { // 必须返回对象
      get() {
        track() // 告诉Vue 这个值需要被"追踪"
        return value
      },
      set(newValue) {
        clearTimeout(timeout) // 防抖动
        timeout = setTimeout(() => {
          value = newValue  // 将新的值赋值，get再调用就返回新的值
          trigger() // 告诉Vue更新页面
        }, delay)
      }
    }
  })
}
export default {
  setup() {
    return {
      text: useDebouncedRef('hello')
    }
  }
}
```
### provide 与 inject
- 实现祖孙组件间的通信
- 父组件使用`provide`提供数据，后代组件使用`inject`使用数据
- 一般隔代传数据使用，父子使用`props`

父组件
```js
setup() {
  ...
  let car = reactive({ name: '路虎', price: '100W' });
  provide('car', car)
}
```
孙组件
```js
setup(props, context) {
  ...
  const car = inject('car');
  return { car }
}
```
### 响应式数据的判断
- `isRef` 是否为ref对象
- `isReactive` 是否由`reactive`创建的响应式代理
- `isReadonly` 是否由`readonly`创建的只读代理
- `isProxy` 是否由`reactive`、`readonly`方法创建的代理，`readonly`创建时不会改变数据类型，还是`Proxy`对象

## Composition API的优势
- Options API 存在的问题
  - 新增、修改一个需求，就需要分别在data、methods、computed里修改
- 组合式 API 的优势
  - 可以将每个功能分别写成hook函数，更加优雅的组织代码，让相关功能的代码有序的组织在一起

## 新组件
### Fragment
- Vue2中组件必须要有根标签
- Vue3中组件可以没有根标签，内部会将多个标签包含在一个 Fragment 虚拟元素中
- 好处：可以减少标签层级和内存占用

### Teleport
传送标签，能将组件的HTML结构移动到指定位置
```vue
<template>
  <!--to 是目标位置，也可以是id选择器-->
  <teleport to="body">
    <div v-if="isShow" class="mask">
      <div class="dialog">
        <h3>弹窗</h3>
      </div>
    </div>
  </teleport>
</template>`
```
### Suspense
悬疑的意思，等待异步组件时渲染的一些额外内容，有更好的用户体验

使用步骤
  - 异步引入组件
```js
import { defineAsyncComponent } from 'vue';
const Child = defineAsyncComponent(() => import('./Child.vue'));
```
  - 使用`suspense`包裹组件，并配置好`default`和`fallback`
```vue
<template>
  <div class="app">
    <suspense>
      <template #default>
        <Child />
      </template>
      <template #fallback>
        <h4>加载中...</h4>
      </template>
    </suspense>
  </div>
</template>


<!--Child.vue-->
<template>
  <div class="app">
    <h3>{{num}}</h3>
  </div>
</template>
<script>
export default {
  setup() {
    let num = ref(0);
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve({num})
      }, 3000)
    })
  }
  
  // 使用async
  async setup() {
    let num = ref(0);
    const p = new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve({num})
      }, 3000)
    })
    return await p;
  }
}
</script>
```

