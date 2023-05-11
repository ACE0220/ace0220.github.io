---
title: vue项目的基础设施搭建（一）
date: 2021-02-05 18:02:30
categories:
  - web
  - infrastructure
  - front-end
tags:
  - web
  - infrastructure
  - front-end
  - 前端
  - vue3
  - typescipt
  - eslint
  - husky
  - pnpm
---

vue3 + typescript + eslint + husky + pnpm

统一化团队的代码风格，样式，git规范等

<!-- more -->

## 概览

一般前端基建包括以下内容：
 - 基本语言：javascript、Typescript
 - 环境配置：env文件内容注入
 - 打包构建工具：例如Webpack、Rollup、vite等；
 - 包管理工具：例如npm、Yarn、pnpm等；
 - 前端框架：例如React、Vue.js等；
 - 代码质量工具：例如ESLint、Prettier等；
 - 单元测试：例如Jest、Mocha等；
 - HTTP请求库：例如Axios、Fetch等；
 - 状态管理工具：例如Redux、Mobx、vuex、pinna等；
 - UI组件库：例如Ant Design、Element UI、自建组件库等；
 - 前端性能分析工具：例如Lighthouse、WebPageTest等；
 - 应用部署与自动化工具：例如Docker、Travis CI等。

## monorepo项目搭建

现在很大一部分框架都采用了pnpm进行多包的管理，例如vue，element-plus等

官方文档：https://pnpm.io/zh/motivation

具体事项不再细说，本文章主要目的是手把手的操作与记录。

### pnpm安装

个人建议pnpm全局安装，更加的方便

```bash
# 全局安装pnpm
npm install -g pnpm
# 打印版本，当前我使用的8.0.0
pnpm --version 
# 列出帮助文档
pnpm -h 
```

### 项目搭建

pnpm-workspace.yaml定义了工作空间的根目录，并能够使您从工作空间中包含 / 排除目录 。默认情况下，包含所有子目录。

在项目根目录初始化和创建pnpm-workspace.yaml文件，并填入示例内容

```bash
pnpm init
```

```bash
touch pnpm-workspace.yaml
```

示例：

```yaml
# pnpm-workspace.yaml
packages:
  # packages下所有直接子包
  - 'packages/*'
```

假定我们的模块分块是core，utils，components，那么就可以在packages目录下分别新建这三个文件夹，分别执行pnpm init去生成对应package.json文件

示例：其他模块同理，这里不再赘述

```bash
cd packages
mkdir core && cd core && pnpm init
```

#### node_modules扁平化的问题

pnpm的node_modules结构是非扁平化的，而npm和yarn采用了平铺的node_modules结构，平铺结构的一个较明显的问题是幽灵依赖，即在package.json中没有定义，但是我们可以导入使用的依赖。

如果需要pnpm将node_modules平铺，根目录新建.npmrc文件，填入shamefully-hoist=true

```sh
# .npmrc
shamefully-hoist=true
```

## typescript

typescript的安装一般会选择跟随项目，即在项目内安装typescript，避免不同typescript版本导致的兼容性问题

### 安装和初始化

全项目ts，安装在根目录即可

```sh
pnpm add typescript -Dw
```

初始化，操作完成后执行目录会生成一个tsconfig.json

```sh
npx tsc --init
```

### tsconfig

参考element-plus，为了提高tsconfig的扩展性，提供tsconfig.base.json，tsconfig.web.json, tsconfig.json等文件，tsconfig.json主要作为一个入口，用于引用其他tsconfig文件


```json
// tsconfig.base.json
{
  "compilerOptions": {
    /* Visit https://aka.ms/tsconfig to read more about this file */
    
    "target": "ES2018", // es版本，不同的es版本会对es特性进行降级
    "module": "ESNext", // 指定生成哪个模块系统代码 "CommonJS" "ES6"或 "ESNext"。 
    "moduleResolution": "node", // 如何处理模块
    "checkJs": false, // 在 .js文件中报告错误。与 --allowJs配合使用
    "allowJs": false, // 允许编译javascript文件
    "outDir": "dist", // 解析非相对模块名的基准目录
    "baseUrl": "./", // 设置baseUrl来告诉编译器到哪里去查找模块。 所有非相对模块导入都会被当做相对于 baseUrl
    "sourceMap": false,
    "strict": true,
    "noUnusedLocals": true, // 若有未使用的局部变量则抛错
    "resolveJsonModule": true, // 允许解析json文件
    "allowSyntheticDefaultImports": true, // 允许从没有设置默认导出的模块中默认导入。这并不影响代码的输出，仅为了类型检查。 设置了esModuleInterop和module !== es2015 / esnext
    /**
    *
    * import * as moment from "moment" 等价于 const moment = require("moment")
    * import moment from "moment" 等价于 const moment = require("moment").default
    * es6模块规范规定，import * as x 应该是一个对象，ts处理成 =require(xxx)的行为是把导入当作一个可调用的函数，不符合规范
    * 开启esModuleInterop会自动修复这个转译问题
    */
    "esModuleInterop": true, 
    "removeComments": false,
    // 如果composite为true
    // rootDir设置，如果没有被显式指定，默认为包含tsconfig文件的目录
    // 必须匹配到include模式或者files数组
    "rootDir": ".", // 所有输入的 非声明文件 中的最长公共路径，
    "types": [], // 指定要包含的类型包名称，而不需要在源文件中引用
    "paths": { // 路径映射
      "@acelcdev/lc-client-*": ["packages/*"]
    }
  }
}

```

```json
// tsconfig.web.json
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": {
    "composite": true, // 引用的工程必须启用新的composite设置
    "jsx": "preserve", // tsx文件里面支持jsx
    "lib": ["ES2018", "DOM", "DOM.Iterable"], // 编译过程中需要引入的库文件的列表
    "types": [],
    "skipLibCheck": true // 忽略所有的声明文件（ *.d.ts）的类型检查。
  },
  "include": ["packages/**/*"],
  "exclude": [
    "node_modules",
    "**/dist",
    "**/__tests__/**/*",
    "**/gulpfile.ts",
    "**/test-helper",
    "**/*.md",
    "docs"
  ]
}
```

总入口

```json
// tsconfig.json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.web.json" }
  ]
}
```

### 测试

项目根目录执行

```json
npx tsc --build tsconfig.json
```

在根目录生成了dist文件夹，内部结构与packages一致，同时具有js文件和d.ts声明文件


## eslint

### 安装和初始化

```sh
pnpm add eslint -Dw
```

```sh
pnpm create @eslint/config
```

这里笔者选择的是To check syntax and find problems

```sh
? How would you like to use ESLint? … 
  To check syntax only
❯ To check syntax and find problems
  To check syntax, find problems, and enforce code style
```

使用的是es6规范，所以选择javascript modules，要看自己项目需求

```sh
? What type of modules does your project use? ...
> JavaScript modules (import/export)
  CommonJS (require/exports)
  None of these
```

看自己项目需求

```sh
? Which framework does your project use? … 
  React
❯ Vue.js
  None of these
```

使用ts?

```
Does your project use TypeScript? » No / Yes
```

两个都选

```
? Where does your code run? ...  (Press <space> to select, <a> to toggle all, <i> to invert selection)
√ Browser
√ Node
```

配置文件格式

```
? What format do you want your config file to be in? ...
> JavaScript
  YAML
  JSON
```

笔者选择了yes

```
eslint-plugin-vue@latest @typescript-eslint/eslint-plugin@latest @typescript-eslint/parser@latest
? Would you like to install them now? › No / Yes
```

后续的可以根据项目需求调整即可，最后生成.eslint.js文件

项目根目录安装下列eslint插件

```sh
pnpm add eslint-plugin-vue @typescript-eslint/eslint-plugin@latest -Dw
```

### .eslintrc.js & .eslintignore

```javascript
// .eslintrc.js
module.exports = {
    "env": {
        "browser": true,
        "es2021": true,
        "node": true
    },
    "extends": [
        "eslint:recommended",
        "plugin:vue/vue3-recommended", // 原来是plugin:vue/vue3-essential
        "plugin:@typescript-eslint/recommended"
    ],
    "overrides": [
    ],
    "parser": "@typescript-eslint/parser",
    "parserOptions": {
        "ecmaVersion": "latest",
        "sourceType": "module"
    },
    "plugins": [
        "vue",
        "@typescript-eslint"
    ],
    "rules": {
        "semi": ["error", "always"],
    }
}
```

.eslintignore文件是一个纯文本文件，每一行都是一个glob模式告知eslint忽略哪些文件或者目录

```sh
# .eslintignore
*.js
dist
docs
```

### 测试

根目录package.json新增脚本，目前只需要校验packages里面的ts文件，具体文件需要根据项目需求进行配置

```json
{
  ...
  "scripts": {
    "lint": "eslint --ext .ts packages/",
    "lint:fix": "eslint --ext .ts packages/ --fix"
  },
  ...
}
```

在index.ts内定义了const a = 1，故意不带分号，eslintrc里面设置了不带分号就报错

```sh
pnpm lint

> eslint --ext .ts packages/

/path/packages/core/index.ts
  1:7   warning  'a' is assigned a value but never used  @typescript-eslint/no-unused-vars
  1:12  error    Missing semicolon                       semi

✖ 2 problems (1 error, 1 warning)
  1 error and 0 warnings potentially fixable with the `--fix` option.
```

这个时候可以运行pnpm lint:fix进行修复

```sh
pnpm lint:fix
# 修复后，只剩一个a变量定义但没有使用的warning
> eslint --ext .ts packages/ --fix

/path/packages/core/index.ts
  1:7  warning  'a' is assigned a value but never used  @typescript-eslint/no-unused-vars

✖ 1 problem (0 errors, 1 warning)
```

### vscode eslint设置

生效的前提是必须去vscode的扩展商店安装eslint插件

```json
// .vscode/settings.json
{
  "update.enableWindowsBackgroundUpdates": false,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  // 每次保存的时候自动格式化
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "dbaeumer.vscode-eslint",
  "eslint.format.enable": true,
  "eslint.alwaysShowStatus": true,
  "eslint.validate": [
    "vue",
    "typescript",
    "typescriptreact"
  ]
}
```

## husky & commitlint

husky是前端工程化的一个重要工具，可以方便的向项目中添加git hooks，一般在commit之前校验代码，commit的时候检验commit信息是否符合规范，而且设置简单。

### husky安装和初始化

```sh
pnpm add husky -Dw
```

package.json中添加prepare脚本，执行git init，再执行pnpm prepare

```json
{
  ...
  "scripts": {
    ...
    "prepare": "husky install"
  },
  ...
}
```

执行完上述命令后，根目录会存在一个.husky文件夹，接下来执行以下命令，在commit的时候，就会自动运行pnpm lint，如果lint不通过，将会终止git commit。

```sh
npx husky add .husky/pre-commit "pnpm lint"
```

如果git commit因为代码校验不通过而被终止，这时候可以通过pnpm lint:fix进行代码自动格式化，通过代码校验后即可以再次commit

### commitlint安装和初始化

commitlint的作用是校验commit信息的规范性，官方提供了对应commit信息的模板，类似于git commit -m "test" 这类的commit是无法通过校验的。

一般要求的格式是

- feat: xxx
- fix: xxx
- docs: xxx

官方文档：https://commitlint.js.org/#/reference-prompt

示例

```git
git commit -m "feat: add some feature"
git commit -m "fix: fix some bug"
```

#### 安装和初始化

安装

```sh
pnpm add @commitlint/cli @commitlint/config-conventional -Dw
```

设置commitlint需要遵循的规范，在根目录创建commitlint.config.js，填入以下内容

```javascript
module.exports = {
  extends: ["@commitlint/config-conventional"]
};
```

配合husky使用

```
npx husky add .husky/commit-msg  'npx --no -- commitlint --edit ${1}'
```

### 测试

首先故意设置const a = 1 不带分号，导致pre-commit终止

```sh
> eslint --ext .ts packages/


/path/packages/client/index.ts
  1:7   warning  'a' is assigned a value but never used  @typescript-eslint/no-unused-vars
  1:14  error    Missing semicolon                       semi

✖ 2 problems (1 error, 1 warning)
  1 error and 0 warnings potentially fixable with the `--fix` option.

 ELIFECYCLE  Command failed with exit code 1.
```

运行pnpm lint:fix修复，修复后提示a未被使用，但是没有error

```sh
> eslint --ext .ts packages/ --fix

/path/packages/client/index.ts
  1:7  warning  'a' is assigned a value but never used  @typescript-eslint/no-unused-vars

✖ 1 problem (0 errors, 1 warning)
```

再次提交commit，故意不按照commit规范

```sh
git add .
git commit -m "test commit-msg hook"
```

此时提示commit-msg hook error

```sh
⧗  input: test commit-msg hook
✖   Please add rules to your `commitlint.config.js`
    - Getting started guide: https://commitlint.js.org/#/?id=getting-started
    - Example config: https://github.com/conventional-changelog/commitlint/blob/master/%40commitlint/config-conventional/index.js [empty-rules]

✖   found 1 problems, 0 warnings
ⓘ   Get help: https://github.com/conventional-changelog/commitlint/#what-is-commitlint

husky - commit-msg hook exited with code 1 (error)
```

修改commit msg，校验通过

```git
git commit -m "test: test commitlint"
```

```sh
feat/initial 6490b0c] test: test commit lint
 3 files changed, 1077 insertions(+)
 create mode 100755 .husky/commit-msg
```

## 总结

本文的内容相对基础，主要是总结一下具体的用途和解决方案，并非很详细的教学内容，如果需要高级的用法，最好的方法还是通过官方文档，去学习如何配置，高级用法等。

下一篇计划开始编写与环境配置、构建相关的内容。





