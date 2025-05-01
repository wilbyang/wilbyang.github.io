---
layout: post
title:  "Phoenix LiveView 的 Stream 功能详解"
date:   2025-04-30 21:16:02 +0200
categories: Elixir
---

LiveView 的 `stream` 功能是 Phoenix 1.7 引入的强大特性，专门用于高效处理大型或频繁更新的列表数据。它通过智能的 DOM 差分更新机制，可以大幅提升列表操作的性能。

### 1. 什么是 Stream？

Stream 提供了一种高效的方式来管理列表数据，特别是当需要：
- 渲染大型列表
- 频繁添加/删除列表项
- 对列表项进行排序或过滤
- 避免全列表重新渲染

传统方式使用 `assign` 渲染列表时，任何变动都会导致整个列表重新渲染。而 Stream 只会更新发生变化的部分。

### 2. 基本用法

#### 2.1 初始化 Stream

在 LiveView 的 `mount` 或事件处理中初始化：

```elixir
def mount(_params, _session, socket) do
  items = [1, 2, 3, 4, 5]
  {:ok, stream(socket, :items, items)}
end
```

#### 2.2 在模板中渲染

使用 `stream` 宏和 `for` 渲染列表：

```elixir
~H"""
<ul id="items">
  <li :for={ {id, item} <- @streams.items} id={id}>
    Item: <%= item %>
  </li>
</ul>
"""
```

注意：
- 每个项会自动分配唯一 DOM ID
- `id={id}` 是必需的，用于高效更新

#### 2.3 添加新项

```elixir
def handle_event("add", _value, socket) do
  new_id = System.unique_integer([:positive])
  new_item = %{id: new_id, name: "Item #{new_id}"}
  {:noreply, stream_insert(socket, :items, new_item, at: 0)}
end
```

#### 2.4 删除项

```elixir
def handle_event("delete", %{"id" => id}, socket) do
  {:noreply, stream_delete(socket, :items, id)}
end
```

### 3. 核心 API

LiveView 提供了多种 Stream 操作函数：

| 函数 | 用途 |
|------|------|
| `stream/3` | 初始化或重置整个 stream |
| `stream_insert/4` | 插入新项 (`at: 0` 开头, `at: -1` 结尾) |
| `stream_delete/3` | 删除指定项 |
| `stream_update/3` | 更新指定项 |
| `stream_put/3` | 插入或更新项 |

### 4. 高级用法

#### 4.1 批量操作

```elixir
socket
|> stream_insert(:items, new_item1, at: 0)
|> stream_insert(:items, new_item2, at: 0)
|> stream_delete(:items, old_item_id)
```

#### 4.2 与 Ecto 结合

```elixir
def handle_event("load_more", _, socket) do
  more_items = Repo.all(from p in Post, limit: 10)
  {:noreply, stream_insert(socket, :posts, more_items, at: -1)}
end
```

#### 4.3 排序和过滤

```elixir
def handle_event("sort", %{"by" => "name"}, socket) do
  sorted = Enum.sort_by(socket.assigns.streams.items, & &1.name)
  {:noreply, stream(socket, :items, sorted)}
end
```

### 5. 性能优势

Stream 通过以下方式优化性能：
1. **最小化 DOM 操作**：只更新变化的项
2. **智能 DOM 复用**：保持不变的项不重新渲染
3. **自动 DOM ID 管理**：无需手动管理 key
4. **高效的补丁更新**：通过 morphdom 进行智能 DOM 比对

### 6. 实际示例：待办事项列表

```elixir
defmodule MyApp.TodoLive do
  use MyAppWeb, :live_view

  def mount(_params, _session, socket) do
    {:ok, stream(socket, :todos, [])}
  end

  def handle_event("add", %{"text" => text}, socket) do
    todo = %{id: System.unique_integer([:positive]), text: text, done: false}
    {:noreply, stream_insert(socket, :todos, todo, at: -1)}
  end

  def handle_event("toggle", %{"id" => id}, socket) do
    todo = Enum.find(socket.assigns.streams.todos, &(&1.id == String.to_integer(id)))
    updated_todo = %{todo | done: !todo.done}
    {:noreply, stream_update(socket, :todos, updated_todo)}
  end

  def handle_event("delete", %{"id" => id}, socket) do
    {:noreply, stream_delete(socket, :todos, String.to_integer(id))}
  end

  def render(assigns) do
    ~H"""
    <div>
      <form phx-submit="add">
        <input type="text" name="text" placeholder="New todo" />
      </form>
    
      <ul id="todos">
        <li :for={ {id, todo} <- @streams.todos} id={id}>
          <input 
            type="checkbox" 
            checked={todo.done} 
            phx-click="toggle" 
            phx-value-id={todo.id}
          />
          <span class={if todo.done, do: "line-through"}>
            <%= todo.text %>
          </span>
          <button phx-click="delete" phx-value-id={todo.id}>Delete</button>
        </li>
      </ul>
    </div>
    """
  end
end
```

### 7. 调试 Stream

1. **检查 assigns**：
   ```elixir
   IO.inspect(socket.assigns.streams)
   ```

2. **浏览器控制台**：
   ```javascript
   // 查看当前 stream 数据
   document.querySelectorAll("[data-phx-stream]")
   ```

3. **DOM 观察**：
   使用浏览器开发者工具观察 DOM 更新情况

### 8. 注意事项

1. 每个 stream 项必须有唯一标识（通常使用 `:id`）
2. 避免在 stream 中存储大型对象
3. 对于分页场景，考虑结合 `stream_insert` 和 `at: -1`
4. 复杂的排序/过滤操作可能需要重置整个 stream

Stream 功能使得处理动态列表变得极其高效，特别是在需要频繁更新的场景下，相比传统的列表渲染方式可以显著提升性能。