---
title: jest基础入门-配置typescript测试
categories:
  - tests
  - jest
tags:
  - tests
  - jest
  - e2e
  - unit test
date: 2021-07-10 16:53:00
---

现代前端基本都是基于typescript开发，测试阶段也要同步使用typescript

<!-- more -->

## demo

github: https://github.com/ACE0220/blog-demos/tree/main/test-framwork/jest-demo

如果对您有帮助，请给个star，谢谢～

## 依赖安装

```sh
pnpm add jest typescript ts-jest @jest/globals ts-node -D
# or
npm install -D jest typescript ts-jest @jest/globals ts-node
```

## tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2018",
    "module": "ES2020",
    "rootDir": "./src",
    "baseUrl": "./src",
    "moduleResolution": "node",
    "esModuleInterop": true
  }
}
```

## jest config file

在jest配置文件中，通过ts-jest预设集将esm转换成commonjs语法

```typescript
// rootdir/jest.config.ts
import type { JestConfigWithTsJest } from 'ts-jest';

const jestConfig: JestConfigWithTsJest = {
  // [...]
  preset: 'ts-jest/presets/default-esm',
  extensionsToTreatAsEsm: ['.ts'],
  moduleNameMapper: {
    '^(\\.{1,2}/.*)\\.js$': '$1',
  },
  testMatch: ['**/tests/*.test.ts'], // 指定任意文件夹下的test文件夹内的*.test.ts文件，才会被jest运行测试
  transform: {
    // '^.+\\.[tj]sx?$' to process js/ts with `ts-jest`
    // '^.+\\.m?[tj]sx?$' to process js/ts/mjs/mts with `ts-jest`
    '^.+\\.tsx?$': [
      'ts-jest',
      {
        useESM: true,
      },
    ],
  },
};

export default jestConfig;

```


## 测试运行

在package.json中增加

```json
// package.json
{
  "scripts": {
    "test": "jest --config jest.config.ts"
  },
}
```

运行测试

```sh
npm run test

> jest-demo@1.0.0 test
> jest --config jest.config.ts

 PASS  tests/sum.test.ts
  sum test
    ✓ add 1 + 2 to equal 3 (1 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        0.688 s, estimated 1 s
Ran all test suites.
```

配置jest的typescript和esm支持就是这么简单，并没有还多复杂的操作，主要还是工具集的使用。
