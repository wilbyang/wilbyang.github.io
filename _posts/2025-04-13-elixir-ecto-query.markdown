---
layout: post
title:  "Elixir Ecto Querying"
date:   2025-04-13 21:16:02 +0200
categories: Elixir
---
# Querying a Database Entity Using a Repository in Elixir



To query a database entity using a repository in the Elixir REPL (`iex -S mix`), you typically use Ecto, which is the standard database wrapper and query generator for Elixir projects. Here’s a step-by-step guide:

## 1. Start the REPL

Make sure you are in your project directory and start the interactive shell:

```sh
iex -S mix
```

## 2. Alias Your Repo and Schema

For convenience, alias your repository and the schema (entity) you want to query. For example, if your repo is `MyApp.Repo` and your schema is `MyApp.User`:

```elixir
alias MyApp.Repo
alias MyApp.User
```

## 3. Import Ecto.Query

Import the `from/2` macro to build queries:

```elixir
import Ecto.Query, only: [from: 2]
```

## 4. Build and Run Queries

You can now build queries using Ecto’s query DSL and execute them with the repository.

**Fetch all users:**

```elixir
Repo.all(User)
```

**Fetch users with a specific condition:**

```elixir
query = from u in User, where: u.age > 18, select: u
Repo.all(query)
```
This will return all users older than 18 as `%User{}` structs[4][5][6].

**Fetch specific fields:**

```elixir
query = from u in User, where: u.email == "alice@example.com", select: u.name
Repo.all(query)
```
This returns a list of names for users with the specified email[2][6].

**Use variables in queries (pin operator):**

```elixir
email = "alice@example.com"
query = from u in User, where: u.email == ^email, select: u
Repo.all(query)
```
The `^` operator pins the value of the variable into the query[2].

## 5. Querying with Raw SQL (Advanced)

If you need to run a raw SQL query:

```elixir
Ecto.Adapters.SQL.query(Repo, "SELECT * FROM users WHERE id = $1", [1])
```
This returns a result map with rows and columns[9].

## 6. Example Session

```elixir
iex> alias MyApp.{Repo, User}
iex> import Ecto.Query, only: [from: 2]
iex> Repo.all(User)
iex> query = from u in User, where: u.age > 18, select: u
iex> Repo.all(query)
iex> email = "alice@example.com"
iex> query = from u in User, where: u.email == ^email, select: u.name
iex> Repo.all(query)
```

## Summary Table

| Task                        | Example Code                                                                 |
|-----------------------------|------------------------------------------------------------------------------|
| Fetch all entities          | `Repo.all(User)`                                                             |
| Query with condition        | `from u in User, where: u.age > 18`                                          |
| Select specific fields      | `from u in User, select: u.name`                                             |
| Use variable in query       | `from u in User, where: u.email == ^email`                                   |
| Run raw SQL                 | `Ecto.Adapters.SQL.query(Repo, "SELECT * FROM users WHERE id = $1",[1])`    |

This approach allows you to interactively query your database entities directly from the Elixir REPL using your repository and Ecto’s query DSL[2][4][6].

Citations:
[1] https://hexdocs.pm/ecto/Ecto.Repo.html
[2] https://elixirschool.com/en/lessons/ecto/querying_basics
[3] https://elixirforum.com/t/handling-a-very-large-database-query-using-streams/46917
[4] https://hexdocs.pm/ecto/Ecto.html
[5] https://blog.appsignal.com/2023/02/14/under-the-hood-of-ecto.html
[6] https://clouddevs.com/elixir/database-interaction/
[7] https://hexdocs.pm/ecto/replicas-and-dynamic-repositories.html
[8] https://www.amberbit.com/blog/2018/6/12/executing_raw_sql_queries_in_elixir/
[9] https://stackoverflow.com/questions/29417162/run-custom-sql-query-with-ecto
[10] https://ludu.co/course/discover-elixir-phoenix/building-a-json-api/
[11] https://stackoverflow.com/questions/27532109/query-repo-using-a-string
[12] https://hexdocs.pm/entity/
[13] https://github.com/josemrb/ecto_ltree
[14] https://github.com/elixir-ecto/ecto
[15] https://elixirforum.com/t/how-to-see-all-tables-in-ecto/57976
[16] https://elixirforum.com/t/how-to-join-multiple-models-in-on-ecto-query/19927
[17] https://loadforge.com/guides/efficient-database-querying-in-phoenix
[18] https://hexdocs.pm/ecto_entity/0.1.1/
[19] https://stackoverflow.com/questions/52685873/compose-ecto-queries-in-elixir
[20] https://elixirschool.com/en/lessons/ecto/basics

---
Answer from Perplexity: pplx.ai/share