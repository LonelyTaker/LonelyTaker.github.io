---
title: ref和reactive对比
date: 2024-10-18 10:19:33
categories: 前端
tags: Vue
---

区别：

* ref可以定义基本数据类型和对象数据类型；reactive可以定义对象数据类型
* ref创建的对象必须使用`.value`
* reactive重新分配一个新对象，会失去响应式（可以使用`Object.assign`去整体替换）

使用原则：

* 若需要一个基本类型的响应式数据，必须使用`ref`
* 若需要一个响应式对象，层级不深，`ref`和`reactive`都可以
* 若需要一个响应式对象，层级较深，推荐使用`reactive`
