---
title: koa异常处理 - 搭建中间件捕获全局异常
date: 2021-03-17 09:29:12
categories:
  - web
  - infrastructure
  - back-end
tags:
  - web
  - infrastructure
  - back-end
  - 后端
  - aop
  - error catcher
---

随着项目的扩大，项目中的try catch会越来越多，项目会更加臃肿。在开始之前，先了解koa的执行多个中间件的洋葱模型。

<!-- more -->


## 洋葱模型

这是网上最常见的关于洋葱模型的示意图

![](/pics/infrastructure/koa-req-res-model.png)

先看一段代码

```typescript
router.get('/', (ctx: Context) => {
    console.log('get route')
    ctx.body = 'welcome'
})

const middleware1 = async (ctx: Context, next: Next) => {
  console.log('mw1 start');
  await next();
  console.log('mw1 end');
}

const middleware2 = async (ctx: Context, next: Next) => {
  console.log('mw2 start');
  await next();
  console.log('mw2 end');
}

app.use(middleware1);
app.use(middleware2);
app.use(router.routes());

app.listen(3000);
console.log('app running in port 3000')
```

运行后打开浏览器访问localhost:3000，打印如下

```sh
mw1 start
mw2 start
get route
mw2 end
mw1 end
```

搭配洋葱模型，不难理解，next的作用到底是什么

**next方法的执行，会指向下一个中间件，等待下一个中间件执行完毕，再返回当前中间件**

```typescript
app.use(middleware1);
app.use(middleware2);
app.use(router.routes());
```

根据这个next方法，就可以理解执行顺序了

1. 执行middleware1，打印mw1 start，遇到next，跳到middleware2
2. 执行middleware2，打印mw2 start，遇到next，跳到router.routes()
3. router.routes()没有next，执行结束，返回上一个中间件middleware2
4. middleware2中next的后面剩余部分执行，打印mw2 end，返回上一个中间件middleware1
5. middleware1中next的后面剩余部分执行，打印mw1 end

**将以上的知识结合到request和response上，可以得出一个答案**

**Koa 的洋葱模型指的是以 next() 函数为分割点，先由外到内执行 Request 的逻辑，再由内到外执行 Response 的逻辑**

### demo

github demo: https://github.com/ACE0220/blog-demos/tree/main/infrastructure/back-end/global-error-aop

运行：pnpm dev:onion

## 简单版本的全局异常捕捉

根据洋葱模型的特性，那么在进入router中间件之前，我们就可以通过try catch包含着next方法，next方法执行的错误就可以捕获到

```typescript
import Koa from 'koa';

export default async function globalException(ctx: Koa.Context, next: Koa.Next) {
  
  try {
    console.log('global exception start');
    await next();
    console.log('global exception end');
  } catch(err: any) {
    const errResult = err as { message: string };
    console.log('global exception capture');
    ctx.body = `Server error: ${ errResult.message }`;
  }
}
```

### demo

github demo: https://github.com/ACE0220/blog-demos/tree/main/infrastructure/back-end/global-error-aop

运行：pnpm dev:ge
