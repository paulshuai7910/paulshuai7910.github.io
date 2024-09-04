---
title: Typescript-tools
date: 2024-04-14 19:35:11
tags:
---

# Pick

Pick<T, K>用于从类型 T 中选取一组属性 K，创建一个新的类型。

```ts
interface Person {
  name: string
  age: number
  address: string
}

type PersonNameAndAge = Pick<Person, "name" | "age">

const personData: PersonNameAndAge = {
  name: "Alice",
  age: 30,
}
```

# Omit

Omit<T, K>用于从类型 T 中排除一组属性 K，创建一个新的类型。

```ts
interface Person {
  name: string
  age: number
  address: string
}

type PersonWithoutAddress = Omit<Person, "address">

const personData: PersonWithoutAddress = {
  name: "Bob",
  age: 25,
}
```

# Partial

Partial<T> 将类型 T 的所有属性变为可选的。

```ts
interface Person {
  name: string
  age: number
  address: string
}

type PartialPerson = Partial<Person>

const partialPersonData: PartialPerson = {
  name: "Alice",
}
```

# Required

Required<T> 将类型 T 的所有属性变为必填的。

```ts
interface Person {
  name?: string
  age?: number
  address?: string
}

type RequiredPerson = Required<Person>

const requiredPersonData: RequiredPerson = {
  name: "Bob",
  age: 30,
  address: "Somewhere",
}
```

# Readonly

Readonly<T> 将类型 T 的所有属性变为只读的。

```ts
interface Person {
  name: string
  age: number
}

type ReadonlyPerson = Readonly<Person>

const readonlyPersonData: ReadonlyPerson = {
  name: "Charlie",
  age: 25,
}

// 以下会报错，因为属性是只读的
readonlyPersonData.name = "David"
```

# Record

Record<K, T> 创建一个对象类型，其属性键为 K 类型，属性值为 T 类型。

```ts
type KeyType = "key1" | "key2" | "key3"
type ValueType = string

type MyRecord = Record<KeyType, ValueType>

const recordData: MyRecord = {
  key1: "value1",
  key2: "value2",
  key3: "value3",
}
```

# Exclude

Exclude<T, U> 从类型 T 中排除可以赋值给类型 U 的类型。

```ts
type Primitive = string | number | boolean
type NonNumber = Exclude<Primitive, number>

const nonNumberData: NonNumber = "stringValue" as NonNumber // 或者 true as NonNumber
```

# Extract

Extract<T, U> 从类型 T 中提取可以赋值给类型 U 的类型。

```ts
type Primitive = string | number | boolean
type Numeric = number | bigint
type NumericPrimitive = Extract<Primitive, Numeric>

const numericPrimitiveData: NumericPrimitive = 10 as NumericPrimitive
```

# ReturnType

ReturnType<T>用于获取函数类型 T 的返回值类型。

```ts
function add(a: number, b: number): number {
  return a + b
}

type AddReturnType = ReturnType<typeof add>

const result: AddReturnType = 5
```
