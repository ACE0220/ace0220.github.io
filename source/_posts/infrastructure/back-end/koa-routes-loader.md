---
title: koa路由自动加载
date: 2021-03-08 14:03:23
categories:
  - web
  - infrastructure
  - backend
tags:
  - web
  - infrastructure
  - backend
  - 后端
  - router
  - loader
---

一般我们在大部分node框架中都见过，在既定的文件夹内只要新增了文件或者文件夹，路由会自动生成，无需人工引入。

这种做法就是我们常说的约定优于配置的一种实现。

<!-- more -->

假定文件夹结构如下，且内部均实现了router.get('/')
-- src
----routes
--------index.ts
--------user
------------info.ts
------------modify.ts

那么理论上就应该生成如下路由

- http://localhost:port
- http://localhost:port/user/info
- http://localhost:port/user/modify

## 开发思路

1. 将路由的初始化等工作交给工具类，工具类初始化返回根路由实例，app可以继续操作路由。
2. 指定特定的路由文件夹，由工具类进行扫描，生成路由返回

## 功能点

1. 工具类单例模式，只需要一个实例即可
2. 初始化配置与koa-router初始化配置一致
3. **核心：递归读取路由文件夹，生成绝对路径**
  3.1 读取模块文件
  3.2 生成路由
  3.3 返回路由

## 功能实现

```typescript
import path from "path";
import fs from "fs";
import Router, { IRouterOptions } from "koa-router";

class RoutesLoader {
  private options: IRouterOptions = {} as IRouterOptions;

  private static instance: RoutesLoader | null = null;

  /**
   * 1. singleton
   * @returns instance
   */
  static getInstance() {
    if (!this.instance) {
      this.instance = new RoutesLoader();
    }
    return this.instance;
  }

  /**
   * 2. The initial configuration is the same as the initial configuration of the koa-router
   * @param options
   * @returns root router
   */
  async init(options?: IRouterOptions) {
    if (options) {
      this.options = options;
    }
    const routeFiles = this.getFiles();
    // 3.3 return root route
    return await this.loadRoutesWrapper(routeFiles);
  }

  /**
   * typescript custom guard
   * Determine whether data is an instance of the Router
   * @param data
   * @returns
   */
  isRouter(data: any): data is Router {
    return data instanceof Router;
  }

  /**
   * generate a root router
   * @returns rootRouter
   */
  setRootRouter() {
    const rootRouer = new Router(this.options);
    return rootRouer;
  }

  /**
   * 3. get files from routes dir, return an array with file absolute path item
   * @returns an array with file absolute path item
   */
  getFiles() {
    // 3.1 Read all routing files recursively
    function _getFiles(dir: string, filepath: string): string[] {
      let retArr: string[] = [];
      const fullPath = path.resolve(dir, filepath);
      const files = fs.readdirSync(fullPath);
      for (const file of files) {
        const stat = fs.statSync(path.resolve(fullPath, file));
        if (stat.isDirectory()) {
          retArr = [...retArr, ..._getFiles(fullPath, file)];
        } else if (stat.isFile()) {
          retArr.push(path.resolve(fullPath, file));
        }
      }
      return retArr;
    }

    return _getFiles(process.cwd(), "src/routes");
  }

  /**
   * 3.2 handle all routes
   * @param allFullFilePath
   * @returns root router after handle all routes
   */
  async loadRoutesWrapper(allFullFilePath: string[]) {
    // get rootRouter
    const rootRouer = this.setRootRouter();

    for (const fullPath of allFullFilePath) {
      const module = await import(fullPath);
      if (this.isRouter(module.default)) {
        rootRouer.use(module.default.routes(), module.default.allowedMethods());
      }
    }
    return rootRouer;
  }
}

export default RoutesLoader.getInstance();

```

## demo

[跳转github打开clone demo运行](https://github.com/ACE0220/blog-demos/tree/main/infrastructure/backend/koa-routes-loader)