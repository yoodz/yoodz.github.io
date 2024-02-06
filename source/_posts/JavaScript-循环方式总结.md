---
title: JavaScript 循环遍历方式总结
tags: JavaScript循环
categories: JavaScript
abbrlink: f0387b05
date: 2019-05-24 10:20:59
---
[参考链接](https://juejin.im/post/5bdfa6a6e51d4520fe63fdd5)

### for
```javascript
  for (let index = 0; index < array.length; index++) {
    console.log(array[index])
  }
```
### Array map()
```javascript
  // value: 当前的元素
  // index：当前元素的下标
  // arr：循环的数组
  array.map((value, index, arr) => {
    console.log(value, index)
  })
```

### Array forEach()
```javascript
  array.forEach((value, index, arr) => {
    console.log(value)
    console.log(index)
    console.log(arr)
  })
```

### for of
```javascript
// 允许遍历 Arrays（数组）, Strings（字符串）, Maps（映射）,
// Sets（集合）等可迭代(Iterable data)的数据结构,注意它的兼容性。
// for of遍历数组时，遍历的是数组元素值
  for (let i of array) {
    console.log(i)
  }
```

### for in
```javascript
  //  for in 循环的时候，不仅遍历自身的属性，还会找到 prototype 上去，所以最好在循环体内加一个判断，
  // 就用 object.hasOwnProperty(i)，这样就避免遍历出太多不需要的属性。
  // for in 应用于数组循环返回的是数组的下标和数组的属性和原型上的方法和属性，而for in应用于对象循环返回的是对象的属性名和原型中的方法和属性
  for (const key in object) {
    if (object.hasOwnProperty(key)) {
      const element = object[key];
      console.log(key, element)
    }
  }
```

### Array reduce()
对数组中的每个元素执行一个由您提供的reducer函数(升序执行)，将其结果汇总为单个返回值。
```javascript
  arr.reduce((x, y) => {
    console.log(x, y)
    return x + y
  })
```

### Array some()
测试数组中是不是至少有1个元素通过了被提供的函数测试。它返回的是一个Boolean类型的
```javascript
  arr.some(x => {
    return x === 1
  })
```

### Array every()
测试一个数组内的所有元素是否都能通过某个指定函数的测试。它返回一个布尔值。
```javascript
  arr.every(x => x > 0)
```

### while
```javascript
  let i = 0
  while(arr[i]) {
    console.log(i, arr[i])
    i++
  }
```

### Array filter()
创建一个新数组, 其包含通过所提供函数实现的测试的所有元素。 
```javascript
  arr.filter( x => x > 20)
```