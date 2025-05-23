---
title: Promise 的实现
date: 2020-10-30T16:59:53+08:00
draft: false
tags:
  - 前端
ShowToc: true
TocOpen: true
ShowWordCount: true
summary: "实现了一个简易的 Promise 类，并对 Promise.all 和 Promise.race 方法进行了自定义实现"
cover:
  image: "/images/Promise 的实现/cover.jpg"
  alt: "cover image"
  caption: "封面图来自 Unsplash | 作者 Alberto Barrera"
  relative: false
---

## Promise 类

```javascript
class Pro {
  callbacks = [];
  state = "pending";
  value = null;
  constructor(fn) {
    // 初始化，把resolve作为参数传入，等待调用
    fn(this.resolve.bind(this));
  }
  // callback为回调，先注册，也就是放入callbacks数组中
  then(callback) {
    if (this.state === "pending") {
      this.callbacks.push(callback);
    } else {
      // 由于state不是pending, 遵循promise状态只能改一次的要求，我们直接操作回调传入参数执行
      callback(this.value);
    }
    return this;
  }
  // resolve也就是fn的第一次参数，循环执行所有callback
  resolve(value) {
    this.state = "fulfilled";
    // setTimeout使内部变成异步，在同步执行完最后执行这里，处理fn是同步的情况下then中的回调函数已经注册，然后在这里去执行，不会出现callbacks是空数组的情况
    setTimeout(() => {
      this.value = value;
      this.callbacks.forEach((callback) => callback(value));
    });
  }
}
```

## 调用

```javascript
new Pro((resolve) => {
  setTimeout(() => {
    console.log(0);
    resolve("resolve");
  }, 2000);
})
  .then((tip) => {
    console.log(1);
    console.log(tip);
  })
  .then((tip) => {
    console.log(2);
    console.log(tip);
  });
```

## Promise.all 实现

```javascript
Promise.prototype.all = function (promises) {
  let results = [];
  let promiseCount = 0;
  let promisesLength = promises.length;
  return new Promise(function (resolve, reject) {
    for (let item of promises) {
      // 执行每个item
      Promise.resolve(item).then(
        function (res) {
          promiseCount++;
          // 按照顺序插入结果
          results[i] = res;
          // 如果全部执行成功，返回成功
          if (promiseCount === promisesLength) {
            return resolve(results);
          }
        },
        function (err) {
          return reject(err);
        }
      );
    }
  });
};
```

## Promise.race 实现

```javascript
Promise.prototype.race = function (promises) {
  return new Promise((resolve, reject) => {
    for (let item of promises) {
      Promise.resolve(item)
        .then((res) => {
          return resolve(res);
        })
        .catch((err) => {
          return reject(err);
        });
    }
  });
};
```

简单来说就是声明 Promise 时，会执行 Promise 第一个函数参数和 then 的参数函数。
`then` 用来把回调传入 callback 数组中，相当于注册，规定好了 reslove 时，回调的执行，然后等待 resolve 调用，resolve 就会把 callback 数组中的函数全部执行

- then 中 `return this`，用于实现 then 的链式调用
- 如果 Promise 是同步的，则执行 resolve 的时候 callback 还没注册
