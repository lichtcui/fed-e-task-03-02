# 一、简答题
#### 1、请简述 Vue 首次渲染的过程。
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

#### 2、请简述 Vue 响应式原理。
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

#### 3、请简述虚拟 DOM 中 Key 的作用和好处。
作用：判断新老节点是否相同
好处：减少 vdom 操作，提高性能

#### 4、请简述 Vue 中模板编译的过程。
- compileToFunctions
  - 先从缓存中加载编译好的 render 函数
  - 缓存中没有调用 compile 开始编译
- compile(template, options)
  - 合并 options
  - 调用 baseCompile 编译模版
- baseCompile(template.trim(), finalOptions)
  - parse()  
    把模版字符串 template 转换成抽象语法树 AST tree  
  - optimize()  
    标记 AST tree 中的静态根节点 sub trees  
    sub trees 不需要在每次重新渲染的时候重新生成节点  
    patch 阶段跳过 sub trees
  - generate()  
    AST tree 生成 js 的创建代码  
- compileToFunctions(template, ...)  
  继续把上一步中生成的字符串形式 js 代码转换为函数  
  createFunction()  
  render 和 staticRenderFns 初始化完毕  
  挂载到 Vue 实例的 options 对应的属性中  