---
title: nest.js 微服务基本入门
categories:
  - microservice
tags:
  - microservice
  - node
  - nest.js
date: 2021-05-07 16:53:00
---

Nest.js支持微服务架构，微服务架构可以将单体结构解耦成多个服务，实现独立部署维护的目的。本文将基于nest.js实现一个微服务架构demo，不会包含所有业务模块的具体实现。

<!-- more -->

# 前置

## github demo

https://github.com/ACE0220/blog-demos/tree/main/microservice/nest-ms

## 简要架构

### 业务架构

初定分为用户模块，商品模块，订单模块。一般设计上来说，商品模块部分功能，订单模块等都要依赖于用户模块。

![](/pics/microservice/intro-nest-1.png)

### 技术架构

nest中内置了几种不同的微服务传输层实现，那么本次采用redis作为消息传输的途径。

- 直接传输：TCP
- 远程过程调度：GRPC
- 消息中转：REDIS、NATS、MQTT、RMQ、KAFKA

简要架构如下

![](/pics/microservice/intro-nest-2.png)

## demo搭建

### pnpm

基于pnpm搭建项目结构, 项目根目录运行pnpm init生成package.json

```sh
pnpm init
```

新建pnpm-workspace.yaml，填入以下内容，说明根目录下的order-module, product-module, user-module下的**子文件夹**才是一个服务，而不是module本身是一个服务

```yaml
packages:
  - 'order-module/*'
  - 'product-module/*'
  - 'user-module/*'
```

为什么不直接使用module作为一个服务，而是module下的才是一个个的服务？

在[<<架构的定义>>这篇文中提过](https://ace0220.github.io/architecture/architecture-defination/#%E6%A8%A1%E5%9D%97%E4%B8%8E%E7%BB%84%E4%BB%B6)，模块是一套一致而相互有密切关联的软件组织，而组件则是自包含，可编程，可重用，与语言无关系的软件单元。就可以理解了，一个模块下，可以有一个或者多个服务（也就是将服务看作是组件）

### 创建module的核心服务

基于上一节中的module和service的结构分层，那么现有阶段我们可以创建每个module的核心服务

建议全局安装nest，便于使用到nest的cli，通过cli去创建。

```sh
npm i -g @nestjs/cli
```

每个module下执行nest new xxx

order-module下执行以下指令，其他模块同理，前期每个module只有一个服务，后期有需要可以直接扩展新的服务，原有服务可以不用移动文件位置。

```sh
nest new order
```

or

```sh
nest new core
```

### 抽取公共依赖到根目录的package.json

每个module下的服务都是通过nest new创建的，基础依赖也是一样的，可以将这些依赖复制到根目录的packge.json

删除所有module下的服务中的node_modules，在根目录执行pnpm install -r

### 微服务改造

#### 更新依赖（可能不需要）

**坑点：@nestjs/microservices需要单独安装，通过nest new xxx生成的代码中的依赖包并不是最新的，与@nestjs/microservices不兼容，所以需要手动更新**

在module下每次通过nest new xxx的时候，都需要操作一次（版本更新可能会不一样）

```sh
pnpm add @nestjs/common@latest @nestjs/core@latest @nestjs/platform-express@latest
pnpm add @nestjs/cli @nestjs/schematics @nestjs/testing -D
```

#### 安装@nestjs/microservices和ioredis

根目录执行以下命令, 一次安装在工作空间，一次安装在服务中，考虑到项目结构后期过大，要保持基础依赖在根目录和服务中都是一致的，编辑器可以不打开工作空间，只打开某个服务进行开发。

```sh
# 在根目录的只需要安装一次就好
pnpm add @nestjs/microservices ioredis -w
# 在nest new 命令创建的文件夹下执行
pnpm add @nestjs/microservices ioredis
```

#### 修改代码

所有服务中的src/main.ts修改成以下形式

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.REDIS,
      options: {
        host: 'localhost',
        port: 6379,
      },
    },
  );
  await app.listen();
}
bootstrap();

```

### 运行测试

分别在order-module/order user-module/user product-module/product 下运行pnpm start:dev

```sh
[10:06:11 PM] File change detected. Starting incremental compilation...

[10:06:11 PM] Found 0 errors. Watching for file changes.

[Nest] 58244  - 05/22/2023, 10:06:12 PM     LOG [NestFactory] Starting Nest application...
[Nest] 58244  - 05/22/2023, 10:06:12 PM     LOG [InstanceLoader] AppModule dependencies initialized +8ms
[Nest] 58244  - 05/22/2023, 10:06:12 PM     LOG [NestMicroservice] Nest microservice successfully started +28ms
```

### 编写e2e测试

nest官方提供了@nestjs/tesing

**记得要先运行user服务**

在user目录下
```sh
pnpm start:dev
```

新建test.app.module.ts

```typescript
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'REDIS_CLIENT',
        transport: Transport.REDIS,
        options: {
          host: 'localhost',
          port: 6379,
        },
      },
    ]),
  ],
})
export class TestAppModule {}

```

新建app.e2e-spec.ts

```typescript
// user-module/user/test/app.e2e.redis.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { TestAppModule } from './test.app.module';

describe('Redis Microservice', () => {
  let appModule: TestingModule;
  let redisClient: any;
  let token: string;
  beforeAll(async () => {
    appModule = await Test.createTestingModule({
      imports: [TestAppModule],
    }).compile();
    redisClient = appModule.get('REDIS_CLIENT');
  });

  it('should return "token" from Redis microservice"', async () => {
    token = await redisClient
      .send({ cmd: 'sign_token' }, { payload: { username: 'admin' } })
      .toPromise();
    console.log(token);
    expect(typeof token).toBe('string');
    expect(token.length).toBeGreaterThan(0);
  });

  it('verify token', async () => {
    const verify = await redisClient
      .send({ cmd: 'verify_token' }, { payload: token })
      .toPromise();
    expect(verify.verify).toBe(true);
  });

  it('verify uncorrect token', async () => {
    const verify = await redisClient
      .send({ cmd: 'verify_token' }, { payload: token + '1' })
      .toPromise();
    expect(verify.verify).toBe(false);
  });

  afterAll(async () => {
    const redisClient = appModule.get('REDIS_CLIENT');
    await redisClient.close();
  });
});


```

测试结果
```sh
pnpm test:e2e

console.log
    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNjg0ODkyNTIyLCJleHAiOjE2ODQ4OTYxMjJ9.0hn2GCXx9H-bTfsfQ8jauIbZOUzBu647Ar300SLyHtk

      at Object.<anonymous> (app.e2e.redis.spec.ts:19:13)

 PASS  test/app.e2e.redis.spec.ts
  Redis Microservice
    ✓ should return "token" from Redis microservice" (26 ms)
    ✓ verfify token (4 ms)
    ✓ verfify uncorrect token (3 ms)

Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
Snapshots:   0 total
Time:        1.314 s, estimated 2 s
Ran all test suites.
```

# 微服务之间进行通信

根据前面的架构图，订单模块，商品模块需要依赖于用户模块，中间是通过redis解耦和互相通信。

我们这里做的只是一个demo，所以将商品模块和订单模块作为redis客户端，申请数据之前向用户模块验证token是否正确。

参考e2e测试里面的test.app.module.ts，商品和订单的app.module.ts可以作这个修改，每个都注入redis客户端

```typescript
// order-module/order/app.module.ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'REDIS_CLIENT',
        transport: Transport.REDIS,
        options: {
          host: 'localhost',
          port: 6379,
        },
      },
    ]),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

## order微服务调用user微服务

我们的需求是在service中，如果token解析返回的username是admin，那么就返回订单列表，反之返回空列表

```typescript
// order-module/order/app.service.ts
import { Inject, Injectable } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Injectable()
export class AppService {
  constructor(
    // 在app.module中注册的redisClient
    // 在service中可以注入，@Inject的参数与app.module中注册redis client的name一致
    @Inject('REDIS_CLIENT')
    private readonly redisClient: ClientProxy,
  ) {}

  async getOrders({ token }): Promise<Array<any>> {
    // 这里调用了user微服务
    const verify = await this.redisClient
      .send({ cmd: 'verify_token' }, { payload: token })
      .toPromise();
    if (verify.verify && verify.decode.username === 'admin') {
      return [
        {
          order_name: 'order1',
        },
        {
          order_name: 'order2',
        },
      ];
    }
    return [];
  }
}
```

## 使用nestjs的混合服务

### 简要架构

更新简要架构，之前的都是微服务之间的调用通过redis，接下来是通过一层bff层，对内调用微服务，对外提供http接口

![](/pics/microservice/intro-nest-3.png)

### 搭建bff层

```sh
nest new bff
```

前面部分有提过，可能需要更新依赖和安装微服务和ioredis的部分，也是一样的。[点击跳转](/microservice/intro-nest/#更新依赖（可能不需要）)

bff的app.module也需要注册redis client，在controller部分就可以调用其他微服务，达到对外提供http接口，对内调用微服务

```typescript
// bff/app.module.ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'REDIS_CLIENT',
        transport: Transport.REDIS,
        options: {
          host: 'localhost',
          port: 6379,
        },
      },
    ]),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}

```

```typescript
import { Controller, Get, Inject, Post, Req } from '@nestjs/common';
import { AppService } from './app.service';
import { ClientRedis } from '@nestjs/microservices';

@Controller()
export class AppController {
  constructor(
    private readonly appService: AppService,
    @Inject('REDIS_CLIENT')
    private readonly redisClient: ClientRedis,
  ) {}

  /**
   * 登录，如果username是admin和不是admin，在获取商品列表的时候是不同的
   * @param request request.body username, password
   * @returns
   */
  @Post('user/login')
  async login(@Req() request: Request): Promise<string> {
    return await this.redisClient
      .send({ cmd: 'sign_token' }, { payload: request.body })
      .toPromise();
  }

  /**
   * 调用user/login,username如果是admin，商品列表有个tag是user，不是admin，tag是random
   * 只是单纯模拟商品列表在不同用户下的列表是不同的
   * @param request 请求
   * @returns 商品列表
   */
  @Get('product/list')
  async product_list(@Req() request: Request) {
    const token = (request.headers as any).token;
    console.log(token);
    return await this.redisClient
      .send({ cmd: 'get_product_list' }, { token })
      .toPromise();
  }

  /**
   * 调用user/login,username如果是admin，订单列表返回空数组，只有用户是admin才能看到
   * 只是单纯模拟订单列表在不同用户下的列表是不同的
   * @param request 请求
   * @returns 订单列表
   */
  @Get('order/list')
  async order_list(@Req() request: Request) {
    const token = (request.headers as any).token;
    return await this.redisClient
      .send({ cmd: 'get_orders' }, { token })
      .toPromise();
  }
}

```

### 使用postman测试

![](/pics/microservice/intro-nest-4.png)

使用token且username是admin，和不使用token或username不是admin返回的数据是不一致的

![](/pics/microservice/intro-nest-5.png)
![](/pics/microservice/intro-nest-6.png)














