---
title: TypeScript 基础

date: 2024-02-09 19:59:03
tags:
---

# 基础类型

```ts
let isFinish: boolean = true
let age: number = 18
let name: string = "张三"
const sym: symbol = Symbol("123")
let ages: Array<number> = [18, 19, 20]
enum Direction {
  NORTH,
  SOUTH,
  EAST,
  WEST,
}
// 枚举类型：支持数字和字符串的枚举常量
let dir: Direction = Direction.NORTH
// 默认情况下，NORTH 的初始值为 0，其余的成员会从 1 开始自动增长。换句话说，Direction.SOUTH 的值为 1，Direction.EAST 的值为 2，Direction.WEST 的值为 3。

// 在 TypeScript 2.4 版本，允许我们使用字符串枚举。
enum Direction {
  NORTH = "NORTH",
  SOUTH = "SOUTH",
  EAST = "EAST",
  WEST = "WEST",
}
```

## Any 类型

在 TypeScript 中，任何类型都可以被归为 any 类型。这让 any 类型成为了类型系统的顶级类型（也被称作全局超级类型）。
any 类型本质上是类型系统的一个逃逸舱。作为开发者，这给了我们很大的自由：TypeScript 允许我们对 any 类型的值执行任何操作，而无需事先执行任何形式的检查。比如：

```ts
let value: any

value.foo.bar // OK
value.trim() // OK
value() // OK
new value() // OK
value[0][1] // OK
```

## Unknown 类型

就像所有类型都可以赋值给 any，所有类型也都可以赋值给 unknown。这使得 unknown 成为 TypeScript 类型系统的另一种顶级类型（另一种是 any）

```ts
let value: unknown

value = true // OK
value = 42 // OK
value = "Hello World" // OK
value = [] // OK
value = {} // OK
```

## Tuple 类型

数组一般由同种类型的值组成，但有时我们需要在单个变量中存储不同类型的值，这时候我们就可以使用元组。在 JavaScript 中是没有元组的，元组是 TypeScript 中特有的类型，其工作方式类似于数组。

```ts
let tupleType: [string, boolean]
tupleType = ["semlinker", true]
```

## Void 类型

某种程度上来说，void 类型像是与 any 类型相反，它表示没有任何类型。当一个函数没有返回值时，你通常会见到其返回值类型是 void：

```js
// 声明函数返回值为void
function warnUser(): void {
  console.log("This is my warning message")
}
```

## Null 和 Undefined 类型

- TypeScript 里，undefined 和 null 两者有各自的类型分别为 undefined 和 null。
- 在 TypeScript 中，null 和 undefined 是所有类型的子类型，也就是说 null 和 undefined 可以赋值给任何类型。

## Never 类型

Never 类型表示的是那些永不存在的值的类型。（即没有任何类型是 Never 的子类型，除了 Never 本身之外）

```ts
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
  throw new Error(message)
}

function infiniteLoop(): never {
  while (true) {}
}
```

# Typescript 断言'<>as'

有时候你会遇到这样的情况，你会比 TypeScript 更了解某个值的详细信息。通常这会发生在你清楚地知道一个实体具有比它现有类型更确切的类型。

通过类型断言这种方式可以告诉编译器，“相信我，我知道自己在干什么”。类型断言好比其他语言里的类型转换，但是不进行特殊的数据检查和解构。它没有运行时没有影响，只是在编译阶段起作用。

类型断言有两种形式：

## 尖括号语法

```ts
let someValue: any = "this is a string"
let strlength: number = (<string>someValue).length
```

## as 语法

```ts
let someValue: any = "this is a string"
let strLength: number = (someValue as string).length
```

## 非空断言 '!'

非空断言（!）是一个方便的工具，用于告诉 TypeScript 编译器某个变量在特定上下文中不可能是 null 或 undefined。然而，滥用非空断言可能会隐藏真正的 null 或 undefined 问题，这些问题在运行时可能会出现，导致应用崩溃。

```ts
const a: number | undefined = undefined
const b: number = a!
console.log(b)
```

# 类型守卫

类型守卫允许开发者在使用特定变量或者属性之前，对变量的类型进行更精准的检查，这在处理联合类型或者任何类型都特别有用，缩小变量的可能范围，提供更具体的类型信息给编译器

类型守卫通常是一个表达式，它返回一个布尔值，用于指示某个值是否属于特定的类型。TypeScript 编译器会利用这个返回值来缩小类型范围，使得在类型守卫之后的代码块中，该值的类型会变得更加精确。

## 类型守卫的种类

TypeScript 中主要有以下几种类型守卫：

- typeof 类型守卫：用于检查一个值是否为原始类型（如 string、number、boolean 等）。
- instanceof 类型守卫：用于检查一个值是否是某个类的实例。
- in 类型守卫：用于检查对象中是否存在某个属性。
- 自定义类型守卫：开发者可以通过编写自定义函数来实现类型守卫，这些函数返回一个布尔值，并通常使用类型谓词（如 isFish）来断言变量的确切类型。

## in 关键字

```ts
interface Admin {
  name: string
  privileges: string[]
}

interface Employee {
  name: string
  startDate: Date
}
type UnknownEmployee = Employee | Admin

function printEmployeeInformation(emp: UnknownEmployee) {
  console.log("Name: " + emp.name)
  if ("privileges" in emp) {
    console.log("Privileges: " + emp.privileges)
  }
  if ("startDate" in emp) {
    console.log("Start Date: " + emp.startDate)
  }
}
```

## typeof 关键字

```ts
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value
  }
  if (typeof padding === "string") {
    return padding + value
  }
  throw new Error(`Expected string or number, got '${padding}'.`)
}
```

# 联合类型

- 定义：联合类型允许一个值可以是几种类型中的一个，即一个变量可以是多个类型中的任何一个，但在任一时间点只能是其中一个类型的具体实例。
- 符号：使用管道符号（|）来表示。

联合类型通常与 null 或 undefined 一起使用：

```ts
const sayHello = (name: string | undefined) => {
  /* ... */
}

let num: 1 | 2 = 1
type EventNames = "click" | "scroll" | "mousemove"
```

# 交叉类型

在 TypeScript 中交叉类型是将多个类型合并为一个类型

- 定义：交叉类型是将多个类型合并成一个新类型，这个新类型拥有所有输入类型的所有属性和方法。即一个值必须同时满足所有合并类型的属性和方法。
- 符号：使用符号&来定义，类似于数学中的“交集”的概念。

```ts
type PartialPointX = { x: number }
type Point = PartialPointX & { y: number }
let point: Point = {
  x: 1,
  y: 1,
}
```

# TypeScript 接口

TypeScript 中的接口是一个非常灵活的概念，除了可用于对类的一部分行为进行抽象以外，也常用于对「对象的形状（Shape）」进行描述。

## 对象的形状

```ts
interface Person {
  name: string
  age: number
}

let semlinker: Person = {
  name: "semlinker",
  age: 33,
}
```

## 可选 | 只读属性

```ts
interface Person {
  readonly name: string
  age?: number
}
```

只读属性用于限制只能在对象刚刚创建的时候修改其值。此外 TypeScript 还提供了 ReadonlyArray<T> 类型，它与 Array<T> 相似，只是把所有可变方法去掉了，因此可以确保数组创建后再也不能被修改。

```ts
let a: number[] = [1, 2, 3, 4]
let ro: ReadonlyArray<number> = a
ro[0] = 12 // error!
ro.push(5) // error!
ro.length = 100 // error!
a = ro // error!
```

## 任意属性

```ts
interface Person {
  name: string
  age?: number
  [propName: string]: any
}
```

## 接口与类型别名的区别

什么是类型别名：

```ts
type MyNumber = number
let num: MyNumber = 10
// 这里定义了一个类型别名 MyNumber，它代表 number 类型。然后可以使用 MyNumber 来声明变量。
```

### 与接口类型不一样，类型别名可以用于一些其他类型，比如原始类型、联合类型和元组

```ts
// primitive
type Name = string

// object
type PartialPointX = { x: number }
type PartialPointY = { y: number }
```

### Extend

接口和类型别名都能够被扩展，但语法有所不同。此外，接口和类型别名不是互斥的。接口可以扩展类型别名，而反过来是不行的。

```ts
Interface extends interface

interface PartialPointX { x: number; }
interface Point extends PartialPointX {
  y: number;
}
Type alias extends type alias
```

# TypeScript 泛型 (<T>)

泛型（Generics）是允许同一个函数接受不同类型参数的一种模板。相比于使用 any 类型，使用泛型来创建可复用的组件要更好，因为泛型会保留参数类型。

```ts
function identity<T>(arg: T): T {
  return arg
}
```

1. 第一个 T 传递类型。
2. 通过链式传递给参数类型和函数返回值类型。
   其中 T 代表 Type，在定义泛型时通常用作第一个类型变量名称。但实际上 T 可以用任何有效名称代替。除了 T 之外，以下是常见泛型变量代表的意思：

K（Key）：表示对象中的键类型；
V（Value）：表示对象中的值类型；
E（Element）：表示元素类型。
其实并不是只能定义一个类型变量，我们可以引入希望定义的任何数量的类型变量。比如我们引入一个新的类型变量 U，用于扩展我们定义的 identity 函数：

```ts
function identity<T, U>(value: T, message: U): T {
  console.log(message)
  return value
}

console.log(identity<Number, string>(68, "Semlinker"))
```

除了为类型变量显式设定值之外，一种更常见的做法是使编译器自动选择这些类型，从而使代码更简洁。我们可以完全省略尖括号，比如：

```ts
function identity<T, U>(value: T, message: U): T {
  console.log(message)
  return value
}

console.log(identity(68, "Semlinker"))
```

## 泛型接口

```ts
interface GenericIdentityFn<T> {
  (arg: T): T
}
```

## 泛型类

```ts
class GenericNumber<T> {
  zeroValue: T
  add: (x: T, y: T) => T
}

let myGenericNumber = new GenericNumber<number>()
myGenericNumber.zeroValue = 0
myGenericNumber.add = function (x, y) {
  return x + y
}
```

## 泛型工具类型

为了方便开发者 TypeScript 内置了一些常用的工具类型，比如 Partial、Required、Readonly、Record 和 ReturnType 等

1. typeof
   在 TypeScript 中，typeof 操作符可以用来获取一个变量声明或对象的类型。

```ts
interface Person {
  name: string
  age: number
}

const sem: Person = { name: "semlinker", age: 33 }
type Sem = typeof sem // -> Person

function toArray(x: number): Array<number> {
  return [x]
}

type Func = typeof toArray // -> (x: number) => number[]
```

2. keyof
   keyof 操作符是在 TypeScript 2.1 版本引入的，该操作符可以用于获取某种类型的所有键，其返回类型是联合类型。

```ts
interface Person {
  name: string
  age: number
}

type K1 = keyof Person // "name" | "age"
type K2 = keyof Person[] // "length" | "toString" | "pop" | "push" | "concat" | "join"
type K3 = keyof { [x: string]: Person } // string | number
```

3. in
   in 用来遍历枚举类型：

```ts
type Keys = "a" | "b" | "c"

type Obj = {
  [p in Keys]: any
} // -> { a: any, b: any, c: any }
```

[typescript](https://segmentfault.com/a/1190000040529237#item-11-41,"ts")
