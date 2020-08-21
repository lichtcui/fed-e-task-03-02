# Virtual DOM
虚拟 DOM(Virtual DOM) 是使用 JavaScript 对象来描述 DOM，虚拟 DOM 的本质就是 JavaScript 对 象，使用 JavaScript 对象来描述 DOM 的结构。应用的各种状态变化首先作用于虚拟 DOM，最终映射 到 DOM。Vue.js 中的虚拟 DOM 借鉴了 Snabbdom，并添加了一些 Vue.js 中的特性，例如:指令和组 件机制。

## 为什么要使用虚拟 DOM
- 使用虚拟 DOM，可以避免用户直接操作 DOM，开发过程关注在业务代码的实现，不需要关注如 何操作 DOM，从而提高开发效率
- 作为一个中间层可以跨平台，除了 Web 平台外，还支持 SSR、Weex
- 关于性能方面，在首次渲染的时候肯定不如直接操作 DOM，因为要维护一层额外的虚拟 DOM， 如果后续有频繁操作 DOM 的操作，这个时候可能会有性能的提升，虚拟 DOM 在更新真实 DOM 之前会通过 Diff 算法对比新旧两个虚拟 DOM 树的差异，最终把差异更新到真实 DOM

#### createElement
createElement() 函数，用来创建虚拟节点 (VNode)，render 函数中的参数 h，就是 createElement()

#### update
内部调用 vm.__patch__() 把虚拟 DOM 转换成真实 DOM

#### patch
对比两个 VNode 的差异，把差异更新到真实 DOM。如果是首次渲染的话，会把真实 DOM 先转换成 VNode

#### createElm
把 VNode 转换成真实 DOM，插入到 DOM 树上

#### patchVnode
比较新旧 VNode 节点，根据不同的状态对dom做合理的更新操作

#### updateChildren
将旧子节点组和新子节点组进行逐一比对，直到遍历完任一子节点组，并使用相关策略
