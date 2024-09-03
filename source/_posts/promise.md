---
title: Promise
date: 2024-02-09 19:59:03
tags:
---

# Promise.all

```js
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return new TypeError(" arugument must be a array")
    }
    let resolvedCounter = 0
    let resolvedNum = promises.length
    let resolvedResult = []
    for (let i = 0; i < resolvedNum; i++) {
      Promise.resolve(promises[i]).then(
        (value) => {
          resolvedCounter++
          resolvedResult[i] = value
          if (resolvedCounter === resolvedNum) {
            return resolve(resolvedResult)
          }
        },
        (error) => {
          return reject(error)
        }
      )
    }
  })
}
```

# Promise.race()

Promise.race() 方法用于在多个 Promise 中选择一个完成的 Promise。它会返回第一个完成的 Promise 的结果。如果所有的 Promise 都没有完成，它会等待所有的 Promise 完成后再返回。

```js
const p = Promise.race([p1, p2, p3])
```

上面代码中，只要 p1、p2、p3 之中有一个实例率先改变状态，p 的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给 p 的回调函数。

# Promise.allSettled()

Promise.allSettled()方法接受一个数组作为参数，数组的每个成员都是一个 Promise 对象，并返回一个新的 Promise 对象。只有等到参数数组的所有 Promise 对象都发生状态变更（不管是 fulfilled 还是 rejected），返回的 Promise 对象才会发生状态变更。

# Promise.any()

只要参数实例有一个变成 fulfilled 状态，包装实例就会变成 fulfilled 状态；如果所有参数实例都变成 rejected 状态，包装实例就会变成 rejected 状态。
