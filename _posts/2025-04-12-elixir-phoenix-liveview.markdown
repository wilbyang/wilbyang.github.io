---
layout: post
title:  使用 Elixir 和 Phoenix LiveView 构建实时应用
date:   2025-04-12 21:16:02 +0200
categories: Elixir
---



Phoenix LiveView 是 Phoenix 框架的一个扩展，允许你构建实时、交互式的用户界面而无需编写 JavaScript。下面我将带你从零开始学习使用 LiveView。

### 1. 环境准备

首先确保你已经安装了 Elixir 和 Phoenix：

```bash
# 检查 Elixir 版本
elixir -v

# 检查 Phoenix 版本
mix phx.new --version
```

如果没有安装，请先安装：

```bash
# 安装 Elixir (根据你的操作系统)
# 例如在 macOS 上：
brew install elixir

# 安装 Phoenix
mix archive.install hex phx_new
```

### 2. 创建新项目

创建一个新的 Phoenix 项目（包含 LiveView）：

```bash
mix phx.new my_live_app --live
cd my_live_app
```

### 3. 基本 LiveView 结构

一个基本的 LiveView 模块通常包含以下几个部分：

```elixir
defmodule HandsDirty2Web.Live.Counter do
  use HandsDirty2Web, :live_view
  @topic "counter"

  def mount(_params, _session, socket) do
    if connected?(socket), do: HandsDirty2Web.Endpoint.subscribe(@topic)

    {:ok, assign(socket, count: 0)}
  end

  def handle_event("increment", _, socket) do
    new_count = socket.assigns.count + 1
    HandsDirty2Web.Endpoint.broadcast_from(self(), @topic, "update", %{count: new_count})
    {:noreply, assign(socket, count: new_count)}
  end

  def handle_event("decrement", _, socket) do
    new_count = socket.assigns.count - 1
    HandsDirty2Web.Endpoint.broadcast_from(self(), @topic, "update", %{count: new_count})
    {:noreply, assign(socket, count: new_count)}
  end

  def handle_info(%{topic: @topic, payload: %{count: new_count}}, socket) do
    {:noreply, assign(socket, count: new_count)}
  end

  def render(assigns) do
    ~L"""
    <div>
      <h1>Count: <%= @count %></h1>
      <button phx-click="decrement">-</button>
      <button phx-click="increment">+</button>
    </div>
    """
  end

end

```

### 4. 路由配置

在 `lib/my_live_app_web/router.ex` 中添加 LiveView 路由：

```elixir
live "/counter", CounterLive
```

### 5. 运行应用

启动 Phoenix 服务器：

```bash
mix phx.server
```

访问 `http://localhost:4000/counter` 你应该能看到计数器应用。

### 6. 核心概念

#### Socket 和状态管理

LiveView 通过 socket 维护状态。`assign/2` 和 `update/3` 是常用的状态管理函数。

#### 事件处理

使用 `phx-click`, `phx-change` 等属性绑定事件：

```elixir
# 模板中
<button phx-click="increment">+</button>

# LiveView 中处理
def handle_event("increment", _, socket) do
  {:noreply, update(socket, :count, &(&1 + 1))}
end
```

#### 实时表单

LiveView 提供了强大的表单支持：

```elixir
~H"""
<.form let={f} for={:user} phx-submit="save">
  <%= text_input f, :name %>
  <%= submit "Save" %>
</.form>
"""

def handle_event("save", %{"user" => user_params}, socket) do
  # 处理表单提交
end
```

### 7. 高级功能

#### PubSub 实时更新

```elixir
# 在某个地方发布消息
Phoenix.PubSub.broadcast(MyApp.PubSub, "topic", {:message, "data"})

# 在 LiveView 中订阅
def mount(_params, _session, socket) do
  if connected?(socket), do: Phoenix.PubSub.subscribe(MyApp.PubSub, "topic")
  {:ok, socket}
end

# 处理消息
def handle_info({:message, data}, socket) do
  {:noreply, assign(socket, message: data)}
end
```

#### 上传文件

LiveView 支持文件上传：

```elixir
def mount(_params, _session, socket) do
  {:ok,
   socket
   |> assign(:uploaded_files, [])
   |> allow_upload(:avatar, accept: ~w(.jpg .jpeg .png), max_entries: 1)}
end

~H"""
<form phx-submit="save" phx-change="validate">
  <.live_file_input upload={@uploads.avatar} />
  <button type="submit">Upload</button>
</form>
"""
```

### 8. 测试 LiveView

Phoenix 提供了 LiveView 测试工具：

```elixir
test "increments count", %{conn: conn} do
  {:ok, view, _html} = live(conn, "/counter")
  assert render(view) =~ "Count: 0"
  
  view |> element("button", "+") |> render_click()
  assert render(view) =~ "Count: 1"
end
```

### 9. 部署

部署 LiveView 应用需要确保 WebSocket 连接正常工作。如果你使用 Phoenix 1.7+，默认配置已经处理好了这一点。

### 学习资源

1. [官方文档](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html)
2. [Phoenix LiveView 课程](https://pragmaticstudio.com/phoenix-liveview)
3. [LiveView 示例应用](https://github.com/chrismccord/phoenix_live_view_example)




