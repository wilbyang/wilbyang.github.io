---
layout: post
title:  "C# .NET Core HowTos"
date:   2025-04-12 21:16:02 +0200
categories: C#
---

### C# .NET Core HowTos
#### 1. **C# .NET Core how to build a todo list web app with user authentication and authorization**

- install .NET Core SDK
- create a new project
```bash
dotnet new webapp -n TodoListApp
```
- navigate to the project directory
```bash
cd TodoListApp
```
- add authentication
```bash
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```
- create a file named `appsettings.json` in the root directory and add the following content
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
- create a file which is for db context and name it `ApplicationDbContext.cs` in the `Data` folder
```csharp
// using will be ignored

public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    public DbSet<TodoItem> TodoItems { get; set; }
}
```
- create a file which is for the todo item model and name it `TodoItem.cs` in the `Models` folder
```csharp
// using will be ignored
public class TodoItem
{
    public int Id { get; set; }
    public string Title { get; set; }
    public bool IsCompleted { get; set; }
}
```
- create a file which is for the todo controller and name it `TodoController.cs` in the `Controllers` folder
```csharp
// using will be ignored
[Route("api/[controller]")]
[ApiController]
public class TodoController : ControllerBase
{
    private readonly ApplicationDbContext _context;

    public TodoController(ApplicationDbContext context)
    {
        _context = context;
    }
    [HttpGet]
    public async Task<ActionResult<IEnumerable<TodoItem>>> GetTodoItems()
    {
        return await _context.TodoItems.ToListAsync();
    }

    [HttpGet("{id}")]
    public async 
}
```







