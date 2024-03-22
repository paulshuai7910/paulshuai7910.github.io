---
title: react的生命周期函数
date: 2024-03-22 22:03:38
tags:
---

生命周期钩子函数应用于有状态组件中，有些只运行一次，有些需要运行多次

创建组件时候需要执行的钩子函数（按照执行顺序）：

```bash
$ hexo new "My New Post"
```

1、constructor：不是 react 的方法，是 es6 的方法,可以在 react 中使用，调用当前组件的父级，也可以在函数中设置当前组件所需要的状态 state；

2、componentWillMount：当前组件即将生成，可以在该函数中修改状态 state；

3、render：创建虚拟 dom，更新 dom，加载配套子组件（并不是加载所有子组件）

4、componentDidMount：组件渲染完毕，执行该函数，在该函数中一定不能更新状态，因为一旦更新状态，就会进行重新 render，又会执行该函数，会陷入死循环。
