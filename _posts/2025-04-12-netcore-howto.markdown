---
layout: post
title:  "C# .NET Core HowTos"
date:   2025-04-12 21:16:02 +0200
categories: C#
---

## C# .NET Core HowTos
### 1. create a new project
- open a terminal and run the following command to create a new web application
```bash
dotnet new webapp -n TodoListApp
```
- navigate to the project directory
```bash
cd TodoListApp
```

### 2. dependency management

```bash
dotnet restore
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.Design --version=8.0.13
dotnet add package Microsoft.EntityFrameworkCore.Relational --version 8.0.13
dotnet add package Pomelo.EntityFrameworkCore.MySql
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

### 3. Storage with Database
`ApplicationDbContextModelSnapshot.cs` 文件会在执行 `dotnet ef migrations add` 命令时**自动更新**。这个文件是 Entity Framework Core 迁移系统的重要组成部分，它的作用如下：

---

#### 关于 `ApplicationDbContextModelSnapshot.cs` 的关键点：

1. **自动维护**：
   - 每次运行 `dotnet ef migrations add <MigrationName>` 时，EF Core 会：
     1. 比较当前模型与快照文件的差异
     2. 生成新的迁移代码（在 `Up()` 和 `Down()` 方法中）
     3. **更新快照文件**以反映当前模型状态

2. **文件作用**：
   - 它是当前数据库模型的"快照"（Snapshot）
   - 用于检测模型变更（下次迁移时对比的依据）
   - 不包含迁移逻辑，只记录模型最终状态

3. **文件位置**：
   ```
   Migrations/
   └── ApplicationDbContextModelSnapshot.cs
   ```

4. **不要手动修改**：
   - 此文件应该完全由 EF Core 工具维护
   - 手动修改可能导致迁移系统混乱

---

#### 示例场景：

假设你修改了 `TodoItem` 模型，添加一个新属性：

```csharp
public class TodoItem
{
    // 原有属性...
    public int Priority { get; set; } // 新增属性
}
```

当你运行：
```bash
dotnet ef migrations add AddPriority
```

EF Core 会：
1. 对比模型与快照文件的差异
2. 生成 `XXXX_AddPriority.cs` 迁移文件（包含 `AddColumn` 操作）
3. **自动更新** `ApplicationDbContextModelSnapshot.cs` 包含新的 `Priority` 属性

---

#### 为什么需要这个文件？

- **迁移幂等性**：确保多次应用迁移不会重复执行
- **模型对比**：检测开发者的模型变更
- **离线工作**：无需连接数据库即可生成迁移

---

#### 如果需要重置？

如果快照文件出现问题（如手动修改导致不一致），可以：

1. 删除所有迁移文件（整个 `Migrations/` 文件夹）
2. 重新运行：
   ```bash
   dotnet ef migrations add InitialCreate
   ```

但要注意这会丢失所有迁移历史，只适用于早期开发阶段。对于生产环境，应该通过创建新的迁移来修复问题。

### 4. Configuration

- configuration: create a file named `appsettings.json` in the root directory and add the following content
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=TodoListApp;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "Jwt": {
    "Key": "your_secret_key",
    "Issuer": "your_issuer",
    "Audience": "your_audience"
  }
}
```



### 5. Project Structure
- create the following folders in the root directory
```
TodoListApp
├── Controllers
├── Data
├── Models
├── Services
├── Migrations
├── wwwroot
└── Views
```


- configure interface and implementation in `Startup.cs`, say `IUserService` and `UserService`
```csharp
public void ConfigureServices(IServiceCollection services)
{

    // scoped means one instance per request
    services.AddScoped<IUserService, UserService>(); 

    // singleton means one instance for the whole application
    // services.AddSingleton<IUserService, UserService>();
    // transient means one instance per injection
    // services.AddTransient<IUserService, UserService>();
    
    
    services.AddControllersWithViews();
}
```
### 6. External Commnunication
- httpclient

- gRPC
```bash
dotnet add package Grpc.AspNetCore.Server.Reflection --version 2.71.0
builder.Services.AddGrpcReflection();
app.MapGrpcReflectionService();

grpcurl -plaintext localhost:5160 list
grpcurl -plaintext localhost:5160 describe greet.Greeter
grpcurl -plaintext -d '{"name": "boyang"}' localhost:5160 greet.Greeter/SayHello
```
- SignalR
```bash
dotnet add package Microsoft.AspNetCore.SignalR --version 8.0.13
```




