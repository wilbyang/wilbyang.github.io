---
layout: post
title:  "C# .NET Core HowTos"
date:   2025-04-12 21:16:02 +0200
categories: C#
---

### C# .NET Core HowTos
#### 1. create a new project
- open a terminal and run the following command to create a new web application
```bash
dotnet new webapp -n TodoListApp
```
- navigate to the project directory
```bash
cd TodoListApp
```
#### 2. dependency management

```bash
dotnet restore
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.Design --version=8.0.13
dotnet add package Microsoft.EntityFrameworkCore.Relational --version 8.0.13
dotnet add package Pomelo.EntityFrameworkCore.MySql
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

#### 3. Configuration

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



#### 4. Project Structure
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

#### 5. Dependency Injection and Configuration
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
    // Add framework services.
    
    services.AddControllersWithViews();
}
```
    




