---
title : '[github-issues]element-plus menu组件隐藏功能'
date : 2023-11-07T14:24:33+08:00
draft : false
tags: ["开源社区"]
---
**起因**：不知道干啥，所以想尝试通过给开源社区贡献微薄之力来提升自己的一些技术水平(做issues，感觉跟刷leetcode似的，不过更有意思一点)

**目标**:修复问题[[issues#14738]](https://github.com/element-plus/element-plus/issues/14738)


**Steps**

1. 一开始无从下手，索性就直接询问chatgpt，而gpt老师给出的解决方案是提供一个collapse-on-click-outside属性，然后使用window函数监听是否点击了menu以外的元素，然后关闭即可。
2. 于是乎就写了这么个监听函数在绑定事件在menu上，但是后面发现出现下拉框的menu是sub-menu组件，原来一开始方向就错了🤡
3. 后来又一看issues，发现已经有人提[pr](https://github.com/element-plus/element-plus/pull/14742)了🤡🤡，不过这也是很好的机会，可以学习一下别人的代码
4. 看了别人写的代码中用到了一个经常听到但是从未去了解过的库，也就是vueuse，发现里面提供了一个已经写好了的工具类，叫做onClickOutside
![](https://github.com/Dmaziyo/blog-img/blob/main/issues14742-2.jpg?raw=true)
![](https://github.com/Dmaziyo/blog-img/blob/main/issue-14738.jpg?raw=true)
5. 我就想着去看看这个onClickOutside的源码长啥样,结果一看，发现看不太明白，但好在通过查看commit历史记录看到了最简单mvp，所以打算学习一下这个utils。
6. [demo](https://code.juejin.cn/pen/7298623517115809830)关键部分在于[event.composedPath()](https://developer.mozilla.org/en-US/docs/Web/API/Event/composedPath),这个api能够返回一个数组，该数据包含了当前事件会被传播到的所有元素。只要判断这个数组是否存在我们目标元素即可。