## 介绍

Vuejs 源码分析在网上的分享有很多，本人也有参考和学习过，依然有一种体会就是，轮子虽多，也不及自己去探究一遍。

本项目的创建旨在个人对 Vuejs 源码的探究和学习（由于本人能力有限，各位大佬看了若有问题的话，还是多提点一下小弟 :see_no_evil: ），当然也可以是作为个人的一个小笔记，然后拿来跟大家一起分享探讨，一起成长。

~~（由于本人时间有限，更新的速度会稍微慢一丢丢，所以分享都会慢慢滴待续～）~~

（现已编写完毕:muscle:）

看本项目之前，我也希望你对 Vuejs 有一定的了解，若有疑问可以看一下官方文档 https://cn.vuejs.org/v2/guide/ 。

<br/>

## 目录

- [【 Vue 源码分析 】数据初始化之响应式探究（上）](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E6%95%B0%E6%8D%AE%E5%88%9D%E5%A7%8B%E5%8C%96%E4%B9%8B%E5%93%8D%E5%BA%94%E5%BC%8F%E6%8E%A2%E7%A9%B6%EF%BC%88%E4%B8%8A%EF%BC%89.md)
- [【 Vue 源码分析 】数据初始化之响应式探究（下）](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E6%95%B0%E6%8D%AE%E5%88%9D%E5%A7%8B%E5%8C%96%E4%B9%8B%E5%93%8D%E5%BA%94%E5%BC%8F%E6%8E%A2%E7%A9%B6%EF%BC%88%E4%B8%8B%EF%BC%89.md)
- [【 Vue 源码分析 】数据初始化之依赖收集（上）](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E6%95%B0%E6%8D%AE%E5%88%9D%E5%A7%8B%E5%8C%96%E4%B9%8B%E4%BE%9D%E8%B5%96%E6%94%B6%E9%9B%86%EF%BC%88%E4%B8%8A%EF%BC%89.md)
- [【 Vue 源码分析 】数据初始化之依赖收集（下）](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E6%95%B0%E6%8D%AE%E5%88%9D%E5%A7%8B%E5%8C%96%E4%B9%8B%E4%BE%9D%E8%B5%96%E6%94%B6%E9%9B%86%EF%BC%88%E4%B8%8B%EF%BC%89.md)
- [【 Vue 源码分析 】数据初始化之依赖更新](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E6%95%B0%E6%8D%AE%E5%88%9D%E5%A7%8B%E5%8C%96%E4%B9%8B%E4%BE%9D%E8%B5%96%E6%9B%B4%E6%96%B0.md)
- [【 Vue 源码分析 】为什么不推荐使用 $forceUpdate](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E4%B8%BA%E4%BB%80%E4%B9%88%E4%B8%8D%E6%8E%A8%E8%8D%90%E4%BD%BF%E7%94%A8%20%24forceUpdate.md)

- [【 Vue 源码分析 】计算属性 Computed](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E8%AE%A1%E7%AE%97%E5%B1%9E%E6%80%A7%20Computed.md)

- [【 Vue 源码分析 】侦听器 Watch](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E4%BE%A6%E5%90%AC%E5%99%A8%20Watch.md)

- [【 Vue 源码分析 】方法 Methods](https://github.com/Andraw-lin/about-Vue/blob/master/docs/【%20Vue%20源码分析%20】方法%20Methods.md)

- [【 Vue 源码分析 】运行机制之 Props](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E8%BF%90%E8%A1%8C%E6%9C%BA%E5%88%B6%E4%B9%8B%20Props.md)

- [【 Vue 源码分析 】混合 Mixin（上）](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E6%B7%B7%E5%90%88%20Mixin%EF%BC%88%E4%B8%8A%EF%BC%89.md)

- [【 Vue 源码分析 】混合 Mixin（下）](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E6%B7%B7%E5%90%88%20Mixin%EF%BC%88%E4%B8%8B%EF%BC%89.md)

- [【 Vue 源码分析 】生命周期 Lifecycle](https://github.com/Andraw-lin/about-Vue/blob/master/docs/【%20Vue%20源码分析%20】生命周期%20Lifecycle.md)

- [【 Vue 源码分析 】异步更新队列之 NextTick](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E5%BC%82%E6%AD%A5%E6%9B%B4%E6%96%B0%E9%98%9F%E5%88%97%E4%B9%8B%20NextTick.md)

- [【 Vue 源码分析 】从 Template 到 DOM 过程是怎样的](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E4%BB%8E%20Template%20%E5%88%B0%20DOM%20%E8%BF%87%E7%A8%8B%E6%98%AF%E6%80%8E%E6%A0%B7%E7%9A%84.md)

- [【 Vue 源码分析 】编译 Compile（上）](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E7%BC%96%E8%AF%91%20Compile%EF%BC%88%E4%B8%8A%EF%BC%89.md)

- [【 Vue 源码分析 】编译 Compile（下）](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E7%BC%96%E8%AF%91%20Compile%EF%BC%88%E4%B8%8B%EF%BC%89.md)

- [【 Vue 源码分析 】渲染 Render (AST -> VNode)](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E6%B8%B2%E6%9F%93%20Render%20(AST%20-%3E%20VNode).md)

- [【 Vue 源码分析 】如何在更新 Patch 中进行 Diff](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E5%A6%82%E4%BD%95%E5%9C%A8%E6%9B%B4%E6%96%B0%20Patch%20%E4%B8%AD%E8%BF%9B%E8%A1%8C%20Diff.md)

<br/>

## 其他话题

- [Object.create(null) 与 {} 究竟有什么不一样](https://github.com/Andraw-lin/about-Vue/blob/master/docs/%E3%80%90%20Vue%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%20%E3%80%91%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%20Lifecycle.md)

<br/>


## 对本项目的疑惑

若各位大佬在看的同时，对本项目有任何想法或者意见，欢迎各位多提点 issue，因为我也想和各位大佬们一起学习和探讨。

另外，若对项目中文章提到的知识点有疑问，我是希望你可以看看 `javascript高级程序设计` 和 `设计模式` 的。因为源码里对 javascript 的基础比较看重，还有就是涉及到的设计模式会很多，所以在探讨过程中还是要慢慢领会和斟酌～

