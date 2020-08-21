# 响应式原理

## 结构
```
src
├─compiler编译相关
├─coreVue 核心库
├─platforms平台相关代码
├─serverSSR，服务端渲染
├─sfc  .vue 文件编译为 js 对象
└─shared公共的代码
```

## 构建版本

- 完整版  
  同时包含编译器和运行时的版本
- 编译器  
  用来将模板字符串编译成为 JavaScript 渲染函数的代码，体积大、效率低
- 运行时  
  用来创建 Vue 实例、渲染并处理虚拟 DOM 等的代码，体积小、效率高。基本上就是除去编译器的代码。
- UMD  
  UMD 版本通用的模块版本，支持多种模块方式  
  vue.js默认文件就是运行时 + 编译器的UMD 版本
- CommonJS  
  CommonJS 版本用来配合老的打包工具比如Browserify或webpack 1
- ES Module  
  从 2.6 开始 Vue 会提供两个 ES Modules (ESM) 构建文件，为现代打包工具提供的版本  
  ESM 格式被设计为可以被静态分析，所以打包工具可以利用这一点来进行“tree-shaking”并将用不到的代码排除出最终的包

## Vue 初始化
`src/core/global-api/index.js` 初始化 Vue 的静态方法
- set / delete / nextTick
- observable
- options  
  初始化 Vue.options 对象  
  并给其扩展 components/directives/filters/_baseVue  
- use 注册 Vue.use() 用来注册插件
- mixin 注册 Vue.mixin() 实现混入
- extend 注册 Vue.extend() 基于传入的 options 返回一个组件的构造函数
- directive / component / filter 注册

`src/core/instance/index.js` 定义 Vue 构造函数，初始化 Vue 实例成员
- 注册 vm 的 _init() 方法，初始化 vm
- 注册 vm 的 $data/$props/$set/$delete/$watch
- 初始化事件相关方法 $on/$once/$off/$emit
- 初始化生命周期相关的混入方法 _update/$forceUpdate/$destroy
- 混入 render $nextTick/_render

## 首次渲染
- Vue 初始化，实例成员，静态成员
- new Vue()
- this._init()
- vm.$mount()
  - src\platforms\web\entry-runtime-with-compiler.js
  - 如果没有传递 render
  - compileToFunctions() 生成 render() 渲染函数
  - options.render = render
  - src\platforms\web\runtime\index.js
  - mountComponent()
- mountComponent(this,el)
  - src\core\instance\lifecycle.js
  - 触发 beforeMount
  - 定义 updateComponent
    - vm._update(vm._render, ...)
    - vm._render() 渲染，渲染虚拟 DOM
    - vm.update() 更新，将 VDom 转换成真实 DOM
  - 创建 Watcher 实例，把 updateComponent 传递
  - 触发 mounted
  - return vm
- watcher.get()
  - 创建完 watcher 会调用一次 get
  - 调用 updateComponent()
  - 调用 vm._render() 创建 VNode
    - 调用 render.call(vm._renderProxy, vm.$createElement)
    - 调用实例化时 Vue 传入的 render()
    - 或者编译 template 生成的 render()
    - 返回 vnode
  - 调用 vm.update(vnode, ...)
    - 调用 vm.__patch__(vm.$el, vnode) 挂载真实 DOM
    - 记录 vm.$el

## 响应式原理
init 方法
- initState() 初始化 Vue 实例的状态
- initData() 向 vue 实例注入 data 属性
- observe() 把 data 对象转换成响应式对象

observe(value) 响应式入口
- 接收 value 参数，响应式要处理的对象
- 判断 value 是否是对象，如果不是对象直接返回
- 判断 value 对象是否有 __ob__，有则说明做过相应式处理直接返回，没有则创建 observer 对象
- 返回 observer 对象

Observer 对象
- 给 value 对象定义不可枚举的 __ob__ 属性，记录当前的 observer 对象
- 数据的响应式处理
  - 设置数组的方法，push pop sort 等，会改变原属组
  - 当这些方法被调用时要发送通知，找到数组对象对应的 observer 并调用 dep.notify() 
  - 遍历数组中的每个成员，对每个成员调用 observe，如果成员是对象，也会转换成响应式对象
- 对象的响应式处理
  - 调用 walk 方法
  - 遍历对象中的所有属性，对每个属性调用 defineReactive

defineReactive
- 为每一个属性创建 dep 对象，让 dep 收集依赖
- 如果当前属性的值是对象，调用 observe (转换成响应式对象)
- 定义 getter
  - 收集依赖，为每个属性收集依赖（包括自对象）
  - 返回属性的值
- 定义 setter
  - 保存新值
  - 如果新值是对象，调用 observe，把新值转换成响应式对象
  - 发送通知，调用 dep.notify()

收集依赖
- 在 watcher 对象的 get 方法中调用 pushTarget 记录 Dep.target 属性
- 访问 data 中的成员的时候收集依赖，defineReactive 的 getter 中收集依赖
- 把属性对应的 watcher 对象添加到 dep 的 subs 数组中
- 给 childOb 收集依赖，目的是子对象添加和删除成员时发送通知

Watcher
- dep.notify() 在调用 watcher 对象的 update() 方法
- queueWatcher() 判断 watcher 是否被处理，如果没有的化添加到 queue 对列中，并调用 flushSchedulerQueue()
- flushSchedulerQueue
  - 触发 beforeUpdate 钩子函数
  - 调用 watcher.run() run() --> get() --> getter() --> updateComponent
  - 清空上一次的依赖
  - 触发 actived 钩子函数
  - 触发 updated 钩子函数

## 实例方法/数据
#### vm.$set
向响应式对象中添加一个属性，并确保这个新属性同样是响应式的，且触发视图更新。它必须用于向响应式对象上添加新属性，因为 Vue 无法探测普通的新增属性 (比如this.myObject.newProperty = 'hi')
> 注意：对象不能是 Vue 实例，或者 Vue 实例的根数据对象。
```js
vm.$set(obj, 'foo', 'test')]
```

#### vm.$delete
删除对象的属性。如果对象是响应式的，确保删除能触发更新视图。这个方法主要用于避开 Vue不能检测到属性被删除的限制，但是你应该很少会使用它。
> 注意：目标对象不能是一个 Vue 实例或 Vue 实例的根数据对象。
```js
vm.$delete(vm.obj, 'msg')
```

#### vm.$watch
观察 Vue 实例变化的一个表达式或计算属性函数。回调函数得到的参数为新值和旧值。表达式只接受监督的键路径。对于更复杂的表达式，用一个函数取代。
- expOrFn：要监视的 $data 中的属性，可以是表达式或函数  
- callback：数据变化后执行的函数  
  函数：回调函数  
  对象：具有 handler 属性(字符串或者函数)，如果该属性为字符串则 methods 中相应的定义  
- options：可选的选项  
  deep：布尔类型，深度监听  
  immediate：布尔类型，是否立即执行一次回调函数  

#### vm.nextTick
Vue 更新 DOM 是异步执行的，批量的在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。
```js
vm.$nextTick(function () {   /* 操作 DOM */  })
Vue.nextTick(function () {})
```
