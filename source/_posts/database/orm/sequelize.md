---
title: Sequelize基本介绍以及使用
categories:
  - database
  - orm
tags:
  - mysql
  - Sequelize
  - orm
date: 2021-06-3 07:53:00
---

Sequelize 是 nest 官方推荐的开源的 orm 工具之一，支持 PostgreSql, MySql, MongoDB, SQL Server, SQLite 等。

<!-- more -->

## demo

github: https://github.com/ACE0220/blog-demos/tree/main/database/orm/sequelize

如果对您有帮助，麻烦给个star， ^_^

## 依赖安装

新建一个项目，新建 package.json，安装所需的依赖

```sh
npm install --save sequelize sequelize-typescript mysql2
npm install --save-dev @types/sequelize
```

创建 tsconfig.json

```json
{
  "compilerOptions": {
    "sourceMap": true,
    "outDir": "dist",
    "strict": true,
    "lib": ["ESNext"],
    "esModuleInterop": true
  }
}
```

## 使用 sequelize 创建数据和查询

## 创建单例 dbutil

在 src 目录下创建 db.ts，对外提供一个单例模式，避免重复创建 sequelize 的实例。

**注意，数据库的配置不会直接放在代码中，一般都是在环境变量中，避免泄密**

```typescript
// src/db.ts
import { Sequelize, Options, Dialect } from "sequelize";

type IPublichSequelizeInitOptions = {
  database: string;
  username: string;
  password: string;
  options: Options & {
    host: string;
    port: number;
    dialect: Dialect;
  };
};

class DBUtil {
  private seq!: Sequelize;
  async auth() {
    try {
      await this.seq.authenticate();
      console.log("Connection success");
    } catch (error) {
      console.log("Unable to connect to the database:", error);
    }
  }

  instance(options?: IPublichSequelizeInitOptions) {
    if (!this.seq && options) {
      const { database, username, password, options: op } = options;
      try {
        this.seq = new Sequelize(database, username, password, op);
      } catch (err: any) {
        throw new Error(err.message);
      }
    }
    return this.seq;
  }

  close() {
    this.seq.close();
  }
}

export const dbutil = new DBUtil().instance({
  database: "seq-test",
  username: "root",
  password: "Aa123456",
  options: {
    host: "localhost",
    port: 3306,
    dialect: "mysql",
  },
});
```

## 创建 models

创建一个用户的模型和商品的模型，创建 model 的时候都需要 sequelize 的实例，封装 dbutil 的作用也是在这里

```typescript
// models/user-model.ts
import { Model } from "sequelize";
import { dbutil } from "../src/db";
import { DataType } from "sequelize-typescript";

export type IPublicUserType = Model<{
  username: string;
}>;

export const UserModel = dbutil.define<IPublicUserType>("User", {
  username: DataType.STRING,
});
```

```typescript
// models/product-model.ts
import { dbutil } from "../src/db";
import { DataType, Model } from "sequelize-typescript";
export type IPublicProductType = Model<{
  product_name: string;
}>;
export const ProductModel = dbutil.define<IPublicProductType>("Product", {
  product_name: DataType.STRING,
});
```

## 向数据库插入和查询数据

```typescript
// src/add.ts
async function main() {
  // 先同步所有模型
  await dbutil.sync();
  // 一次添加一个用户
  await UserModel.create<IPublicUserType>({
    username: "user" + uuidv4(),
  });
  // 一次添加多个商品
  await ProductModel.bulkCreate<IPublicProductType>([
    {
      product_name: "product" + uuidv4(),
    },
    {
      product_name: "product" + uuidv4(),
    },
    {
      product_name: "product" + uuidv4(),
    },
  ]);
}
```

```typescript
// src/query.ts
async function main() {
  const users = await UserModel.findAll<IPublicUserType>();
  const products = await ProductModel.findAll<IPublicProductType>();

  console.log(users.map((item) => item.dataValues));
  console.log(products.map((item) => item.dataValues));
}
```

## 运行测试

使用 ts-node 运行 add.ts 和 query.ts，或者写入 package.json 更方便

```json
"scripts": {
  "add": "npx ts-node ./src/add.ts",
  "query": "npx ts-node ./src/query.ts"
},
```

npm run add之后就可以运行npm run query去查询所有数据

```sh
npm run add

> sequelize@1.0.0 add
> npx ts-node ./src/add.ts

Executing (default): SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE' AND TABLE_NAME = 'Users' AND TABLE_SCHEMA = 'seq-test'
Executing (default): SHOW INDEX FROM `Users`
Executing (default): SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE' AND TABLE_NAME = 'Products' AND TABLE_SCHEMA = 'seq-test'
Executing (default): SHOW INDEX FROM `Products`
Executing (default): INSERT INTO `Users` (`id`,`username`,`createdAt`,`updatedAt`) VALUES (DEFAULT,?,?,?);
Executing (default): INSERT INTO `Products` (`id`,`product_name`,`createdAt`,`updatedAt`) VALUES (NULL,'product1f61beaa-d803-4a23-9222-1a364b85fc0d','2023-05-26 02:57:22','2023-05-26 02:57:22'),(NULL,'product65aead3b-deb6-461e-9d4a-a668ef93c4c5','2023-05-26 02:57:22','2023-05-26 02:57:22'),(NULL,'productc93d5d43-1dc2-4d01-bd7a-7fd83718bed0','2023-05-26 02:57:22','2023-05-26 02:57:22');
```

