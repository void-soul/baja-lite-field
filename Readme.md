# Baja-Lite-Field

[![npm version](https://img.shields.io/npm/v/baja-lite-field.svg)](https://www.npmjs.com/package/baja-lite-field)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

TypeScript 实体字段装饰器库，为 ORM 提供元数据支持。

## 📋 目录

- [特性](#特性)
- [架构](#架构)
- [安装](#安装)
- [快速开始](#快速开始)
- [API 文档](#api-文档)
- [高级用法](#高级用法)

## ✨ 特性

- 🎯 **类型安全**: 完整的 TypeScript 类型支持
- 🔧 **装饰器驱动**: 使用装饰器定义字段元数据
- 🗄️ **多数据库支持**: MySQL, PostgreSQL, SQLite
- 🔄 **自动转换**: 数据库类型与 TypeScript 类型互转
- 📝 **字段验证**: 支持必填、长度、格式等验证
- 🔑 **主键支持**: 支持单主键和复合主键
- 📊 **索引管理**: 自动生成索引定义
- 🔒 **逻辑删除**: 内置逻辑删除支持
- 🎨 **枚举支持**: 枚举类型映射
- 📅 **日期处理**: 自动日期格式转换

## 🏗️ 架构

```
┌─────────────────────────────────────────────────────────┐
│                    Application Layer                     │
│                   (Entity Definitions)                   │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                  @Field Decorator                        │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Collect Metadata via reflect-metadata           │   │
│  │  - Field Name & Type                             │   │
│  │  - Database Column Definition                    │   │
│  │  - Validation Rules                              │   │
│  │  - Conversion Functions                          │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                  Metadata Storage                        │
│  ┌──────────────┬──────────────┬──────────────┐        │
│  │   _fields    │   _columns   │    _ids      │        │
│  │  Field Map   │ Column Names │  Primary Keys│        │
│  └──────────────┴──────────────┴──────────────┘        │
│  ┌──────────────┬──────────────┬──────────────┐        │
│  │   _index     │    _def      │  _logicIds   │        │
│  │ Index Fields │ Default Vals │ Logic Delete │        │
│  └──────────────┴──────────────┴──────────────┘        │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                  ORM Layer (baja-lite)                   │
│  Uses metadata for:                                      │
│  - SQL Generation                                        │
│  - Data Validation                                       │
│  - Type Conversion                                       │
│  - Schema Migration                                      │
└─────────────────────────────────────────────────────────┘
```

### 核心组件

1. **@Field 装饰器**
   - 收集字段元数据
   - 生成数据库列定义
   - 注册转换函数

2. **AField 类**
   - 字段元数据容器
   - 提供 SQL 生成方法
   - 管理数据转换

3. **元数据符号**
   - `_fields`: 字段映射
   - `_columns`: 列名数组
   - `_ids`: 主键字段
   - `_index`: 索引字段
   - `_def`: 默认值

## 📦 安装

```bash
npm install baja-lite-field reflect-metadata
```

## 🚀 快速开始

### 1. 启用装饰器

`tsconfig.json`:

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

### 2. 导入 reflect-metadata

```typescript
import 'reflect-metadata';
```

### 3. 定义实体

```typescript
import { Field } from 'baja-lite-field';

export class User {
  @Field({
    type: 'String',
    P: 'id',
    id: true,
    uuid: true,
    comment: '用户ID'
  })
  id: string;

  @Field({
    type: 'String',
    P: 'username',
    length: 50,
    notNull: true,
    comment: '用户名'
  })
  username: string;

  @Field({
    type: 'String',
    P: 'email',
    length: 100,
    comment: '邮箱'
  })
  email: string;

  @Field({
    type: 'Number',
    P: 'age',
    comment: '年龄'
  })
  age: number;

  @Field({
    type: 'Number',
    P: 'status',
    def: 1,
    comment: '状态'
  })
  status: number;

  @Field({
    type: 'Date',
    P: 'created_at',
    comment: '创建时间'
  })
  createdAt: Date;
}
```

## 📚 API 文档

### @Field 装饰器

```typescript
@Field(options: FieldOption)
```

#### FieldOption 配置

```typescript
interface FieldOption {
  // 基础配置
  type: FieldType;              // 字段类型
  P: string;                    // 属性名（必填）
  C?: string;                   // 列名（默认与 P 相同）
  comment?: string;             // 注释
  
  // 主键配置
  id?: boolean;                 // 是否主键
  uuid?: boolean;               // 是否 UUID（自动生成）
  uuidShort?: boolean;          // 是否短 UUID
  autoIncrement?: boolean;      // 是否自增
  
  // 约束配置
  notNull?: boolean;            // 是否非空
  unique?: boolean;             // 是否唯一
  length?: number;              // 长度限制
  precision?: number;           // 数值精度
  scale?: number;               // 数值小数位
  
  // 默认值
  def?: any;                    // 默认值
  
  // 索引
  index?: boolean;              // 是否索引
  
  // 逻辑删除
  logicDelete?: boolean;        // 是否逻辑删除字段
  deleteState?: string | number; // 删除状态值
  
  // 枚举
  enum?: string;                // 枚举名称
  
  // 导出
  exportable?: boolean;         // 是否可导出（默认 true）
  
  // 转换函数
  Data2SQL?: (data: any) => any;  // 数据 -> SQL
  SQL2Data?: (sql: any) => any;   // SQL -> 数据
}
```

### FieldType 类型

```typescript
type FieldType = 
  | 'String'      // 字符串
  | 'Text'        // 长文本
  | 'Number'      // 数字
  | 'Boolean'     // 布尔
  | 'Date'        // 日期
  | 'DateTime'    // 日期时间
  | 'JSON'        // JSON
  | 'Blob'        // 二进制
  | 'Decimal';    // 精确小数
```

### 数据库类型映射

#### MySQL

```typescript
String      -> VARCHAR(length) / VARCHAR(255)
Text        -> TEXT
Number      -> INT / BIGINT
Boolean     -> TINYINT(1)
Date        -> DATE
DateTime    -> DATETIME
JSON        -> JSON
Blob        -> BLOB
Decimal     -> DECIMAL(precision, scale)
```

#### PostgreSQL

```typescript
String      -> VARCHAR(length) / VARCHAR(255)
Text        -> TEXT
Number      -> INTEGER / BIGINT
Boolean     -> BOOLEAN
Date        -> DATE
DateTime    -> TIMESTAMP
JSON        -> JSONB
Blob        -> BYTEA
Decimal     -> NUMERIC(precision, scale)
```

#### SQLite

```typescript
String      -> TEXT
Text        -> TEXT
Number      -> INTEGER
Boolean     -> INTEGER
Date        -> TEXT
DateTime    -> TEXT
JSON        -> TEXT
Blob        -> BLOB
Decimal     -> REAL
```

## 🔧 高级用法

### 1. 复合主键

```typescript
export class OrderItem {
  @Field({
    type: 'String',
    P: 'order_id',
    id: true,
    comment: '订单ID'
  })
  orderId: string;

  @Field({
    type: 'String',
    P: 'product_id',
    id: true,
    comment: '商品ID'
  })
  productId: string;

  @Field({
    type: 'Number',
    P: 'quantity',
    comment: '数量'
  })
  quantity: number;
}
```

### 2. 逻辑删除

```typescript
export class Article {
  @Field({
    type: 'String',
    P: 'id',
    id: true,
    uuid: true
  })
  id: string;

  @Field({
    type: 'String',
    P: 'title',
    comment: '标题'
  })
  title: string;

  @Field({
    type: 'Number',
    P: 'status',
    logicDelete: true,
    deleteState: 0,  // 0 表示已删除
    def: 1,          // 1 表示正常
    comment: '状态'
  })
  status: number;
}
```

### 3. 枚举字段

```typescript
// 定义枚举
export enum UserRole {
  Admin = 'admin',
  User = 'user',
  Guest = 'guest'
}

export class User {
  @Field({
    type: 'String',
    P: 'role',
    enum: 'UserRole',
    def: UserRole.User,
    comment: '角色'
  })
  role: UserRole;
}

// 注册枚举
import { registerEnum } from 'baja-lite-field';

registerEnum('UserRole', {
  admin: '管理员',
  user: '普通用户',
  guest: '访客'
});
```

### 4. 数据转换

```typescript
export class Product {
  @Field({
    type: 'String',
    P: 'price',
    comment: '价格',
    // 存储时转换为分
    Data2SQL: (yuan) => Math.round(yuan * 100),
    // 读取时转换为元
    SQL2Data: (fen) => fen / 100
  })
  price: number;

  @Field({
    type: 'JSON',
    P: 'tags',
    comment: '标签',
    // JSON 序列化
    Data2SQL: (arr) => JSON.stringify(arr),
    // JSON 反序列化
    SQL2Data: (str) => JSON.parse(str || '[]')
  })
  tags: string[];
}
```

### 5. 自定义列名

```typescript
export class User {
  @Field({
    type: 'String',
    P: 'userId',        // TypeScript 属性名（驼峰）
    C: 'user_id',       // 数据库列名（下划线）
    id: true
  })
  userId: string;

  @Field({
    type: 'String',
    P: 'userName',
    C: 'user_name'
  })
  userName: string;
}
```

### 6. 精确小数

```typescript
export class Order {
  @Field({
    type: 'Decimal',
    P: 'total_amount',
    precision: 10,      // 总位数
    scale: 2,           // 小数位数
    comment: '总金额'
  })
  totalAmount: number;
}
```

### 7. 索引配置

```typescript
export class User {
  @Field({
    type: 'String',
    P: 'email',
    index: true,        // 创建索引
    unique: true,       // 唯一索引
    comment: '邮箱'
  })
  email: string;

  @Field({
    type: 'String',
    P: 'username',
    index: true,
    comment: '用户名'
  })
  username: string;
}
```

### 8. 日期字段

```typescript
export class Article {
  @Field({
    type: 'Date',
    P: 'publish_date',
    comment: '发布日期'
  })
  publishDate: Date;

  @Field({
    type: 'DateTime',
    P: 'created_at',
    comment: '创建时间'
  })
  createdAt: Date;

  @Field({
    type: 'DateTime',
    P: 'updated_at',
    comment: '更新时间'
  })
  updatedAt: Date;
}
```

### 9. JSON 字段

```typescript
export class Config {
  @Field({
    type: 'JSON',
    P: 'settings',
    comment: '配置信息'
  })
  settings: {
    theme: string;
    language: string;
    notifications: boolean;
  };

  @Field({
    type: 'JSON',
    P: 'metadata',
    comment: '元数据'
  })
  metadata: Record<string, any>;
}
```

### 10. 二进制字段

```typescript
export class File {
  @Field({
    type: 'String',
    P: 'id',
    id: true,
    uuid: true
  })
  id: string;

  @Field({
    type: 'String',
    P: 'filename',
    comment: '文件名'
  })
  filename: string;

  @Field({
    type: 'Blob',
    P: 'content',
    comment: '文件内容'
  })
  content: Buffer;
}
```

## 🔍 元数据访问

### 获取字段元数据

```typescript
import { _fields, _columns, _ids } from 'baja-lite-field';
import { Reflect } from 'reflect-metadata';

const user = new User();

// 获取所有字段
const fields = Reflect.getMetadata(_fields, user);
// { id: AField, username: AField, ... }

// 获取列名数组
const columns = Reflect.getMetadata(_columns, user);
// ['id', 'username', 'email', ...]

// 获取主键字段
const ids = Reflect.getMetadata(_ids, user);
// ['id']
```

### AField 方法

```typescript
const field = fields['username'];

// 获取列名
field.C2();  // 'username'

// 获取列名（带别名）
field.C3();  // 'username username'

// 生成 MySQL 列定义
field.Mysql();  // 'username VARCHAR(50) NOT NULL COMMENT "用户名"'

// 生成 SQLite 列定义
field.Sqlite();  // 'username TEXT NOT NULL'

// 生成 PostgreSQL 列定义
field.Postgresql();  // 'username VARCHAR(50) NOT NULL'
```

## 📝 最佳实践

### 1. 实体组织

```typescript
// entities/base.entity.ts
export abstract class BaseEntity {
  @Field({
    type: 'String',
    P: 'id',
    id: true,
    uuid: true,
    comment: 'ID'
  })
  id: string;

  @Field({
    type: 'DateTime',
    P: 'created_at',
    comment: '创建时间'
  })
  createdAt: Date;

  @Field({
    type: 'DateTime',
    P: 'updated_at',
    comment: '更新时间'
  })
  updatedAt: Date;
}

// entities/user.entity.ts
export class User extends BaseEntity {
  @Field({
    type: 'String',
    P: 'username',
    length: 50,
    notNull: true,
    index: true,
    comment: '用户名'
  })
  username: string;
}
```

### 2. 命名规范

```typescript
// 推荐：使用驼峰命名属性，下划线命名列
export class User {
  @Field({
    type: 'String',
    P: 'userId',      // 驼峰
    C: 'user_id',     // 下划线
    id: true
  })
  userId: string;
}
```

### 3. 类型安全

```typescript
// 使用 TypeScript 类型
export class User {
  @Field({
    type: 'String',
    P: 'role',
    enum: 'UserRole'
  })
  role: UserRole;  // 使用枚举类型

  @Field({
    type: 'JSON',
    P: 'settings'
  })
  settings: UserSettings;  // 使用接口类型
}

interface UserSettings {
  theme: 'light' | 'dark';
  language: string;
}
```

### 4. 默认值

```typescript
export class Article {
  @Field({
    type: 'Number',
    P: 'view_count',
    def: 0,  // 设置默认值
    comment: '浏览次数'
  })
  viewCount: number;

  @Field({
    type: 'Number',
    P: 'status',
    def: 1,
    comment: '状态'
  })
  status: number;
}
```

## 🔗 与 baja-lite 集成

```typescript
import { DB, SqlService } from 'baja-lite';
import { Field } from 'baja-lite-field';

// 1. 定义实体
export class User {
  @Field({ type: 'String', P: 'id', id: true, uuid: true })
  id: string;

  @Field({ type: 'String', P: 'username' })
  username: string;
}

// 2. 创建服务
@DB({
  tableName: 'user',
  clz: User,  // 传入实体类
  dbType: DBType.Mysql
})
export class UserService extends SqlService<User> {
  // baja-lite 会自动读取 User 的字段元数据
}

// 3. 使用
const service = new UserService();
await service.insert({ data: { username: 'john' } });
```

## 📄 License

MIT

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📮 联系

- GitHub: [void-soul/baja-lite-field](https://github.com/void-soul/baja-lite-field)
- NPM: [baja-lite-field](https://www.npmjs.com/package/baja-lite-field)
