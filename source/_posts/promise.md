---
title: Promise
date: 2024-02-09 19:59:03
tags:
---

ES6 规定，Promise 对象是一个构造函数，用来生成 Promise 实例。

```js
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

在 **ES6** 中，`Promise` 是用于处理异步操作的一种机制，它允许你更加清晰地编写异步代码。`Promise` 对象有多个方法来处理异步操作，常用的方法包括：

### 1. **`Promise.resolve()`**

`Promise.resolve()` 返回一个已解决（fulfilled）状态的 Promise 对象，且其值为传入的参数。

- 如果传入的参数是一个 Promise，`Promise.resolve()` 会返回该 Promise。
- 如果传入的是一个非 Promise 的值，`Promise.resolve()` 会返回一个已解决的 Promise，并将这个值作为该 Promise 的返回值。

#### 示例：

```javascript
let p1 = Promise.resolve(42)
p1.then((value) => {
  console.log(value) // 输出: 42
})
```

- 如果传入的是一个 `thenable`（即一个有 `then` 方法的对象），它也会被处理为一个 Promise。

#### 示例：

```javascript
let p2 = Promise.resolve({
  then: function (resolve, reject) {
    resolve(100)
  },
})
p2.then((value) => {
  console.log(value) // 输出: 100
})
```

### 2. **`Promise.reject()`**

`Promise.reject()` 返回一个已拒绝（rejected）状态的 Promise 对象，且其拒绝原因为传入的参数。

#### 示例：

```javascript
let p3 = Promise.reject(new Error("Something went wrong"))
p3.catch((error) => {
  console.log(error) // 输出: Error: Something went wrong
})
```

### 3. **`Promise.all()`**

`Promise.all()` 方法接受一个包含多个 Promise 的可迭代对象（通常是一个数组），并返回一个新的 Promise。这个新的 Promise 会等待所有传入的 Promise 都解决后再被解决。如果其中一个 Promise 被拒绝，`Promise.all()` 会立即拒绝。

- 如果所有 Promise 都成功，返回一个包含每个 Promise 返回值的数组。
- 如果其中一个 Promise 失败，返回第一个失败的错误。

#### 示例：

```javascript
let p4 = Promise.all([
  Promise.resolve(1),
  Promise.resolve(2),
  Promise.resolve(3),
])
p4.then((values) => {
  console.log(values) // 输出: [1, 2, 3]
})
```

如果其中一个 Promise 被拒绝：

```javascript
let p5 = Promise.all([
  Promise.resolve(1),
  Promise.reject("Error"),
  Promise.resolve(3),
])
p5.catch((error) => {
  console.log(error) // 输出: Error
})
```

### 4. **`Promise.allSettled()`** (ES2020)

`Promise.allSettled()` 方法接受一个包含多个 Promise 的可迭代对象，并返回一个新的 Promise，这个新的 Promise 会等待所有传入的 Promise 都完成，不论它们是成功还是失败。返回的结果是每个 Promise 的状态（fulfilled 或 rejected）和对应的值或拒绝原因。

- 返回的数组中，每个元素都是一个对象，包含 `status` 和 `value` 或 `reason` 属性。

#### 示例：

```javascript
let p6 = Promise.allSettled([
  Promise.resolve(1),
  Promise.reject("Error"),
  Promise.resolve(3),
])
p6.then((results) => {
  console.log(results)
  // 输出:
  // [
  //   { status: "fulfilled", value: 1 },
  //   { status: "rejected", reason: "Error" },
  //   { status: "fulfilled", value: 3 }
  // ]
})
```

### 5. **`Promise.race()`**

`Promise.race()` 方法接受一个包含多个 Promise 的可迭代对象，并返回一个新的 Promise。这个新的 Promise 会在传入的 Promise 中 **第一个解决或拒绝** 时解决，并返回该 Promise 的值或拒绝原因。

#### 示例：

```javascript
let p7 = Promise.race([
  new Promise((resolve) => setTimeout(resolve, 100, "one")),
  new Promise((resolve) => setTimeout(resolve, 200, "two")),
  new Promise((resolve) => setTimeout(resolve, 50, "three")),
])

p7.then((value) => {
  console.log(value) // 输出: 'three' (因为它是最先解决的)
})
```

### 6. **`Promise.any()`** (ES2021)

`Promise.any()` 方法接受一个包含多个 Promise 的可迭代对象，并返回一个新的 Promise。这个新的 Promise 会在传入的 Promise 中 **第一个成功** 时解决。如果所有传入的 Promise 都被拒绝，返回一个 `AggregateError`（表示多个错误）。

#### 示例：

```javascript
let p8 = Promise.any([
  Promise.reject("Error 1"),
  Promise.resolve("Success 2"),
  Promise.resolve("Success 3"),
])

p8.then((value) => {
  console.log(value) // 输出: "Success 2" (因为它是第一个成功的 Promise)
})
```

如果所有 Promise 都被拒绝：

```javascript
let p9 = Promise.any([Promise.reject("Error 1"), Promise.reject("Error 2")])

p9.catch((error) => {
  console.log(error) // 输出: AggregateError: All promises were rejected
})
```

### 7. **`Promise.finally()`**

`Promise.finally()` 方法接受一个回调函数，这个回调函数会在 Promise 结束时（无论是解决还是拒绝）执行。`finally()` 不会影响 Promise 的最终结果，返回的 Promise 保持原有的状态。

#### 示例：

```javascript
let p10 = Promise.resolve("Success")

p10
  .finally(() => {
    console.log("Promise has been settled") // 不管成功还是失败，都会执行
  })
  .then((value) => {
    console.log(value) // 输出: Success
  })
```

`finally()` 常用于做清理工作，比如隐藏加载提示、关闭数据库连接等。

---

### 总结

ES6 的 `Promise` 对象提供了许多方法来处理异步操作，包括：

- `Promise.resolve()`：返回已解决的 Promise。
- `Promise.reject()`：返回已拒绝的 Promise。
- `Promise.all()`：等待多个 Promise 都成功并返回所有结果，或者在任何一个失败时拒绝。
- `Promise.allSettled()`：等待所有 Promise 完成（无论成功还是失败），返回每个 Promise 的状态。
- `Promise.race()`：返回第一个解决或拒绝的 Promise。
- `Promise.any()`：返回第一个成功的 Promise，所有 Promise 都失败时返回 `AggregateError`。
- `Promise.finally()`：无论 Promise 成功还是失败，都会执行的回调。

# 实现 Promise.all

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
