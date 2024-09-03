---
title: Async await
date: 2024-06-08 13:34:08
tags:
---

# async/await 的原理

在 ES6 中，async/await 是一种用于异步编程的语法糖，它基于 Promise 实现，使得异步代码看起来更像同步代码，从而提高了代码的可读性和可维护性。
async 函数：
一个函数被标记为 async 后，它会自动返回一个 Promise。如果函数内部返回一个值，这个 Promise 会被 resolved 为这个值；如果函数内部抛出一个错误，这个 Promise 会被 rejected 为这个错误。
例如：

```js
async function myAsyncFunction() {
  return "Hello, world!"
}

myAsyncFunction().then((result) => console.log(result)) // 'Hello, world!'
```

await 表达式：
await 表达式用于等待一个 Promise 的结果。如果 Promise 被 resolve，await 表达式会返回 Promise 的值；如果 Promise 被 reject，await 表达式会抛出 Promise 的错误。这个错误可以被 try/catch 块捕获。
例如：

```js
async function myAsyncFunction() {
  const result = await Promise.resolve("Hello, world!")
  return result
}

myAsyncFunction().then((result) => console.log(result)) // 'Hello, world!'
```

# async/await 与 Promise 的区别

## 语法和可读性：

Promise：使用 .then() 和 .catch() 方法来处理异步操作的结果和错误。代码可能会变得嵌套，尤其是在处理多个异步操作时，可读性较差。
async/await：使用类似同步代码的语法，使异步代码更易于理解和维护。可以使用 try/catch 块来处理错误，就像处理同步代码中的错误一样。
例如：

```js
// Promise 方式
function fetchData() {
  return fetch("https://api.example.com/data")
    .then((response) => response.json())
    .then((data) => console.log(data))
    .catch((error) => console.error(error))
}

// async/await 方式
async function fetchData() {
  try {
    const response = await fetch("https://api.example.com/data")
    const data = await response.json()
    console.log(data)
  } catch (error) {
    console.error(error)
  }
}
```

## async/await 的工作模式

async/await 是一个异步执行器，它会在遇到 await 表达式时暂停执行，等待 Promise 的结果。当 Promise 被 resolve 时，async/await 会继续执行，直到下一个 await 表达式或函数返回。
例如：

```js
async function myAsyncFunction() {
  console.log("Start")
  const result = await Promise.resolve("Hello, world!")
  console.log("End")
  return result
}

myAsyncFunction().then((result) => console.log(result)) // 'Start' 'End' 'Hello, world!'
```

在这个例子中，async/await 会在遇到 await Promise.resolve("Hello, world!") 时暂停执行，直到 Promise 被 resolve 为 "Hello, world!"，然后继续执行。

## 错误处理

Promise：错误处理需要通过 .catch() 方法进行链式调用，如果一个 Promise 链中的某个 Promise 被 rejected，错误会沿着链传递，直到被 .catch() 方法捕获。
async/await：可以使用 try/catch 块来集中处理异步操作中的错误，更加直观和方便。
调试：
Promise：在调试时，由于异步操作的嵌套和链式调用，可能会比较复杂。
async/await：由于代码看起来更像同步代码，调试起来相对容易。
