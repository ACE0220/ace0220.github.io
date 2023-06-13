---
title: 手写Promise
categories:
  - javascript
tags:
  - web
  - javascript
  - async
date: 2020-10-01 14:09:00
---

通过手写简易Promise了解基本的promise规范

<!-- more -->

## 核心知识点
  - 初始化 & 异步调用
  - 状态不可变，只能从pending转化到fulfilled或者rejected
  - then & catch & 链式调用
  - 实现静态api：resolve，reject，all, race

## 手写前期思路

  1. 创建promise需要用new，可以采用es6的class
  2. new MyPromise需要传入一个executor函数，executor函数的两个参数是resolve和reject函数，用来改变promise的状态
  3. promise的状态有三种，pending，fulfilled和rejected，调用resolve函数，状态从pending -> fulfilled, 调用reject，状态从pending -> rejected
  4. then方法和catch方法可以链式调用，但是promise的状态只能改变一次，说明返回的是一个新的promise实例，并且考虑catch是不是then的一个语法糖
  5. all和race的思路，参数都是数组，那么是否涉及到数组循环，然后判断的问题？
  6. static resolve 和 static reject，与实例方法then中调用的流程一致，并且也支持链式调用，即返回新的实例同时可以调用实例方法中then的resolve 或 reject

## 初始化 & 异步调用 & 状态不可变 & 链式调用

```javascript
class MyPromise {
  state = 'pending'; // 状态 pending，fulfilled， rejected
  value = undefined; // 成功的值
  reason = undefined; // 失败原因
  resolvedCbs = []; // pending状态下存储onfulfilled函数
  rejectedCbs = []; // pending状态存储onrejected函数

  constructor(executor) {
    /**
     * 
     * @param {*} value 外部调用resolve的值
     */
    const resolveHandler = (value) => {
      // 知识点2，状态不可变
      if(this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
        this.resolvedCbs.forEach(fn => fn(this.value)); // 异步调用，状态会延迟才变化，所以需要提前存好resolvedCallbacks，等到状态改变再依次执行
      }
    }
    /**
     * 
     * @param {*} reason 外部调用reject的原因
     */
    const rejectHandler = (reason) => {
      // 知识点2，状态不可变
      if(this.state === 'pending') {
        this.state = 'rejected';
        this.reason = reason;
        this.rejectedCbs.forEach(fn => fn(this.reason)) // 异步调用，状态会延迟才变化，所以需要提前存好rejectedCallbacks，等到状态改变再依次执行
      }
    }
    /**
     * 使用try catch，避免传入的executor参数执行错误
     */
    try {
      executor(resolveHandler, rejectHandler)
    } catch(err) {
      rejectHandler(err);
    }
  }
  then(onresolved, onrejected) {
    // 参数判断，提高容错
    onfulfilled = typeof onfulfilled === 'function' ? onfulfilled : (v) => v;
    onrejected = typeof onrejected === 'function' ? onrejected : (e) => e;

    // 如果是pending，上述两个函数会被存储
    if(this.state === 'pending') {
      return new MyPromise((resolve, reject) => {
        this.resolvedCbs.push(() => {
          try {
            const newVal = onfulfilled(this.value);
            resolve(newVal);
          } catch(err) {
            reject(err);
          }
        });
        this.rejectedCbs.push(() => {
          try {
            const newReason = onrejected(this.reason);
            reject(newReason)
          } catch(err) {
            reject(err)
          }
        });
      })
    }

    // 如果是同步调用，则马上执行
    if(this.state === 'fulfilled') {
      return new MyPromise((resolve, reject) => {
        try {
          const newVal = onfulfilled(this.value);
          resolve(newVal);
        } catch(err) {
          reject(err);
        }
      })
    }

    if(this.state === 'rejected') {
      return new MyPromise((resolve, reject) => {
        try {
          const newReason = onrejected(this.reason);
          reject(newReason);
        } catch(err) {
          reject(err);
        }
      })
    }
  }
  /**
   * 上文说过考虑catch是不是then的一个语法糖
   */
  catch(onrejected) {
    return this.then(null, onrejected)
  }


}
```

```javascript
// 同步执行
const p = new MyPromise((resolve, reject) => {
  resolve(100);
}).then(res => {
  console.log(res); // 100
}, err => {
  console.log(err); // 不执行
})

// 异步执行
const p1 = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolve(100);
  }, 1000)
}).then(res => {
  console.log(res); // wait 1 second, 100
}, err => {
  console.log(err); // 不执行
})
```

## 实现静态api：resolve，reject，all, race

```javascript
class MyPromise{
  // ... 代码省略

  static resolve(value) {
    return new MyPromise((resolve, reject) => resolve(value))
  }

  static reject(reason) {
    return new MyPromise((resolve, reject) => reject(reason))
  }

  /**
   * promiseList是一个数组，前置工作都是通过循环promiseList
   * all的主要做法是通过一个计数器去统计所有promise是否完成，如果有一个未完成，直接reject结束
   * race的主要做法通过一个标志位去标记是否有一个完成了，如果有，直接resolve
   */
  static all(promiseList = []) {
    const p1 = new MyPromise((resolve, reject) => {
      const result = [];
      const length = promiseList.length;
      let resolvedCount = 0; // 用于计数
      promiseList.forEach(p => {
        p.then(data => {
          result.push(data);
          // 只有在then被执行，才说明是执行成功
          resolvedCount++;
          if(resolvedCount === length) {
            resolve(result)
          }
        }).catch(err => {
          reject(err)
        })
      })
    })
    return p1;
  }

  static race(promiseList = []) {
    let resolved = false; // 标记是否完成了一个
    const p1 = new MyPromise((resolve, reject) => {
      promiseList.forEach(p => {
        p.then(data => {
          if(!resolved) {
            resolve(data);
            resolved = true;
          }
        }).catch(err => {
          reject(err)
        })
      })
    })
    return p1;
  }
}
```