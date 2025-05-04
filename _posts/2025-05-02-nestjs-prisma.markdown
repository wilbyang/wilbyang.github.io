---
layout: post
title:  "nestjs prisma"
date:   2025-05-02 21:16:02 +0200
categories: NestJS
---

Prisma 的查询语法设计遵循了 **类型安全（Type Safety）**、**链式调用（Fluent API）** 和 **直观表达（Declarative Syntax）** 的理念。要理解它的规律，可以从以下几个方面入手：

---

## **1. 核心设计理念**
### **(1) 类型安全（Type Safety）**
Prisma 的所有查询都是 **基于 Schema 自动推导类型** 的，这意味着：
- 查询的字段必须在 Schema 中定义，否则 TypeScript 会报错
- 查询条件必须符合字段类型（如 `Int` 不能和 `String` 比较）
- 关联查询会自动推断返回的嵌套结构

**示例：**
```ts
// ✅ 正确（`email` 在 User 模型中定义）
await prisma.user.findUnique({
  where: { email: "alice@prisma.io" }
});

// ❌ 错误（`username` 未定义，TypeScript 直接报错）
await prisma.user.findUnique({
  where: { username: "alice" } // 类型错误！
});
```

---

### **(2) 链式调用（Fluent API）**
Prisma 的查询方法可以 **链式组合**，类似 SQL 的 `SELECT ... WHERE ... ORDER BY ...`：
```ts
const users = await prisma.user
  .findMany({
    where: { role: "ADMIN" },  // 条件
    orderBy: { name: "asc" },  // 排序
    take: 10,                  // 限制数量
  });
```

---

### **(3) 直观表达（Declarative Syntax）**
Prisma 的查询语法尽量接近自然语言，例如：
- `findUnique` → 查找唯一记录（如主键或唯一索引）
- `findMany` → 查找多条记录
- `create` → 插入数据
- `update` → 更新数据
- `delete` → 删除数据

---

## **2. 查询模式规律**
### **(1) CRUD 操作**
| 操作 | 方法 | 示例 |
|------|------|------|
| **查询单条** | `.findUnique()` | `prisma.user.findUnique({ where: { id: 1 } })` |
| **查询多条** | `.findMany()` | `prisma.user.findMany({ where: { age: { gt: 18 } } })` |
| **创建** | `.create()` | `prisma.user.create({ data: { name: "Alice" } })` |
| **更新** | `.update()` | `prisma.user.update({ where: { id: 1 }, data: { name: "Bob" } })` |
| **删除** | `.delete()` | `prisma.user.delete({ where: { id: 1 } })` |

---

### **(2) 条件查询（WHERE）**
Prisma 的 `where` 使用 **嵌套对象** 表示条件：
```ts
await prisma.user.findMany({
  where: {
    email: { contains: "@gmail.com" },  // 模糊查询
    age: { gt: 18, lt: 30 },           // 范围查询
    OR: [                              // 逻辑 OR
      { name: "Alice" },
      { name: "Bob" }
    ]
  }
});
```

**支持的运算符：**
| 运算符 | 含义 | 示例 |
|--------|------|------|
| `equals` | 等于 | `{ age: { equals: 25 } }` |
| `gt`, `gte` | 大于（或等于） | `{ age: { gt: 18 } }` |
| `lt`, `lte` | 小于（或等于） | `{ age: { lt: 30 } }` |
| `in` | 在列表中 | `{ role: { in: ["ADMIN", "USER"] } }` |
| `contains` | 包含字符串 | `{ email: { contains: "@gmail.com" } }` |
| `startsWith`, `endsWith` | 开头/结尾匹配 | `{ name: { startsWith: "A" } }` |

---

### **(3) 关联查询（JOIN）**
Prisma 的关联查询使用 **嵌套 `include` 或 `select`**：
```ts
// 查询 User 并包含其所有 Post
const userWithPosts = await prisma.user.findUnique({
  where: { id: 1 },
  include: { posts: true },  // 关联查询
});

// 只选择特定关联字段
const userWithPostTitles = await prisma.user.findUnique({
  where: { id: 1 },
  select: {
    name: true,
    posts: { select: { title: true } }  // 只查询 Post 的 title
  }
});
```

---

### **(4) 分页 & 排序**
```ts
// 分页（skip + take）
await prisma.user.findMany({
  skip: 10,  // 跳过前 10 条
  take: 5,   // 取 5 条
});

// 排序（orderBy）
await prisma.user.findMany({
  orderBy: [
    { age: "asc" },   // 按年龄升序
    { name: "desc" }  // 再按名字降序
  ]
});
```

---

## **3. 高级查询模式**
### **(1) 事务（Transaction）**
```ts
// 批量操作（Prisma 自动包装成事务）
const [deletedUser, updatedPost] = await prisma.$transaction([
  prisma.user.delete({ where: { id: 1 } }),
  prisma.post.update({
    where: { id: 2 },
    data: { views: { increment: 1 } }
  })
]);
```

### **(2) 原生 SQL 查询**
```ts
// 直接执行 SQL（适用于复杂查询）
const users = await prisma.$queryRaw`
  SELECT * FROM "User" WHERE "age" > 18
`;
```

---

## **4. 总结：Prisma 查询语法规律**
1. **类型安全**：所有查询都基于 Schema 定义，TypeScript 自动检查。
2. **链式调用**：方法可组合（`findMany` + `where` + `orderBy`）。
3. **直观表达**：方法名接近自然语言（`findUnique`, `create`, `update`）。
4. **嵌套结构**：
   - `where` 使用对象表示条件
   - `include` 或 `select` 处理关联查询
5. **分页/排序**：`skip` + `take` + `orderBy` 实现分页逻辑。

---

### **学习建议**
- **先理解 Schema**：Prisma 的查询能力完全依赖 `schema.prisma` 的定义。
- **多用 TypeScript 提示**：VSCode 的自动补全会告诉你可用的查询选项。
- **从简单查询开始**：先掌握 `findMany` + `where`，再逐步学习关联查询和事务。

通过这种方式，你可以更自然地掌握 Prisma 的查询模式。