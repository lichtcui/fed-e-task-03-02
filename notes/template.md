# 模版编译
目的
- 将模版 template 转换为渲染函数 render  
作用
- Vue 2.x 使用 VNode 描述视图以及各种交互，用户自己编写 VNode 比较复杂
- 用户只需要编写类似 HTML 的代码 - Vue 模板，通过编译器将模板转换为返回 VNode 的 render 函数
- .vue 文件会被 webpack 在构建的过程中转换成 render 函数

## 模版编译过程
#### 解析 parse
解析器将模板解析为抽象语树 AST，只有将模板解析成 AST 后，才能基于它做优化或者生成代码
字符串

#### 优化 optimize
优化抽象语法树，检测子节点中是否是纯静态节点  
一旦检测到纯静态节点  
- 提升为常量，重新渲染的时候不在重新创建节点
- 在 patch 的时候直接跳过静态子树

#### 生成 generate
把字符串转换成函数

## 组件化机制
- 组件化可以让我们方便的把页面拆分成多个可重用的组件
- 组件是独立的，系统内可重用，组件之间可以嵌套
- 有了组件可以像搭积木一样开发网页
 
#### 组件创建和挂载
- 创建根组件，首次 _render() 时，会得到整棵树的 VNode 结构
- 整体流程:new Vue() --> $mount() --> vm._render() --> createElement() --> createComponent()
- 创建组件的 VNode，初始化组件的 hook 钩子函数

#### 组件实例的创建和挂载过程
Vue._update() --> patch() --> createElm() --> createComponent()
