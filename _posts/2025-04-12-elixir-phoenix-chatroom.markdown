---
layout: post
title:  "Elixir Phoenix Chatroom"
date:   2025-04-12 21:16:02 +0200
categories: Elixir
---

### Elixir Phoenix Chatroom

#### 1. **Elixir Phoenix Chatroom**
- **Elixir**: A functional programming language built on the Erlang VM, known for its concurrency and fault-tolerance capabilities.
- **Phoenix**: A web framework for Elixir, designed for building scalable and maintainable applications.
- **Chatroom**: A real-time application that allows users to send and receive messages instantly.
- **Real-time Communication**: Achieved using WebSockets, allowing for two-way communication between the client and server.
- **Channels**: A feature in Phoenix that provides a way to handle real-time communication, allowing multiple users to join and interact in a chatroom.

#### 2. **steps**
- **Install Elixir and Phoenix**: Ensure you have Elixir and Phoenix installed on your machine.

```
mix local.hex
mix archive.install hex phx_new
```
- **Create a new Phoenix project**: Use the `mix phx.new` command to create a new Phoenix application.

```
mix phx.new chatroom
cd chatroom
```
- **Install dependencies**: Run `mix deps.get` to install the required dependencies for your Phoenix application.

```
mix deps.get
```
- **Generate a new channel**: Use the `mix phx.gen.channel` command to create a new channel for the chatroom.

```
mix phx.gen.channel ChatRoom
```
- **Update the router**: Open `lib/chatroom_web/router.ex` and add the following line to the `scope` block:

```elixir
  socket "/socket", ChatroomWeb.UserSocket
```
- **Create a new socket**: Create a new file `lib/chatroom_web/channels/user_socket.ex` and define the socket for the chatroom.

```elixir
defmodule ChatroomWeb.UserSocket do
  use Phoenix.Socket

  ## Channels
  channel "chat_room:*", ChatroomWeb.ChatRoomChannel

  # Transports
  transport :websocket, Phoenix.Transports.WebSocket

  def connect(_params, socket) do
    {:ok, socket}
  end

  def id(_socket), do: nil
end
```
- **Create a new channel**: Create a new file `lib/chatroom_web/channels/chat_room_channel.ex` and define the channel for the chatroom.

```elixir
defmodule ChatroomWeb.ChatRoomChannel do
  use ChatroomWeb, :channel
  def join("chat_room:lobby", _message, socket) do
    send(self(), :after_join)
    {:ok, socket}
  end
  def handle_info(:after_join, socket) do
    push(socket, "user_joined", %{message: "Welcome to the chatroom!"})
    {:noreply, socket}
  end
  def handle_in("new_message", %{"message" => message}, socket) do
    broadcast(socket, "new_message", %{message: message})
    {:noreply, socket}
  end
  def handle_in("user_joined", %{"username" => username}, socket) do
    broadcast(socket, "user_joined", %{message: "#{username} has joined the chatroom!"})
    {:noreply, socket}
  end
  def handle_in("user_left", %{"username" => username}, socket) do
    broadcast(socket, "user_left", %{message: "#{username} has left the chatroom!"})
    {:noreply, socket}
  end
  def handle_in("typing", %{"username" => username}, socket) do
    broadcast(socket, "typing", %{message: "#{username} is typing..."})
    {:noreply, socket}
  end
  def handle_in("stop_typing", %{"username" => username}, socket) do
    broadcast(socket, "stop_typing", %{message: "#{username} stopped typing."})
    {:noreply, socket}
  end
  def handle_in("disconnect", _message, socket) do
    {:stop, :normal, socket}
  end
  def terminate(_reason, _socket) do
    :ok
  end
end
```
- **Create a new JavaScript file**: Create a new file `assets/js/chatroom.js` and define the JavaScript code for the chatroom.

```javascript
import { Socket } from "phoenix"
import { LiveSocket } from "phoenix_live_view"
import { csrfToken } from "./csrf"
import { Hooks } from "./hooks"

let socket = new Socket("/socket", { params: { _csrf_token: csrfToken } })
socket.connect()
let channel = socket.channel("chat_room:lobby", {})

let chatInput = document.getElementById("chat-input")
let chatButton = document.getElementById("chat-button")
let chatMessages = document.getElementById("chat-messages")
let typingIndicator = document.getElementById("typing-indicator")
let username = prompt("Enter your username:")
channel.join()
  .receive("ok", resp => { console.log("Joined successfully", resp) })
  .receive("error", resp => { console.log("Unable to join", resp) })

chatButton.addEventListener("click", () => {
    let message = chatInput.value
    channel.push("new_message", { message: message })
    chatInput.value = ""
    })
channel.on("new_message", payload => {
    let messageItem = document.createElement("li")
    messageItem.innerText = payload.message
    chatMessages.appendChild(messageItem)
    chatMessages.scrollTop = chatMessages.scrollHeight
})
channel.on("user_joined", payload => {
    let messageItem = document.createElement("li")
    messageItem.innerText = payload.message
    chatMessages.appendChild(messageItem)
    chatMessages.scrollTop = chatMessages.scrollHeight
})

channel.on("user_left", payload => {
    let messageItem = document.createElement("li")
    messageItem.innerText = payload.message
    chatMessages.appendChild(messageItem)
    chatMessages.scrollTop = chatMessages.scrollHeight
})
channel.on("typing", payload => {
    typingIndicator.innerText = payload.message
})
channel.on("stop_typing", payload => {
    typingIndicator.innerText = ""
})
chatInput.addEventListener("input", () => {
    channel.push("typing", { username: username })
})
chatInput.addEventListener("blur", () => {
    channel.push("stop_typing", { username: username })
})
chatInput.addEventListener("focus", () => {
    channel.push("user_joined", { username: username })
})
chatInput.addEventListener("blur", () => {
    channel.push("user_left", { username: username })
})
channel.onClose(() => {
    channel.push("disconnect", {})
})
```
- **Update the HTML file**: Open `assets/index.html.eex` and add the following lines to include the chatroom JavaScript file.

```html
<script src="/js/chatroom.js"></script>
```
- **Run the Phoenix server**: Use the `mix phx.server` command to start the Phoenix server.

```
mix phx.server
```
- **Open the chatroom in your browser**: Navigate to `http://localhost:4000` in your web browser to access the chatroom.
- **Test the chatroom**: Open multiple browser tabs and test the chatroom functionality by sending messages, joining, and leaving the chatroom.
- **Deploy the chatroom**: Once you are satisfied with the chatroom functionality, you can deploy it to a production server using tools like Docker or Heroku.
- **Monitor the chatroom**: Use tools like Phoenix LiveDashboard to monitor the performance and health of your chatroom application.
- **Scale the chatroom**: If needed, you can scale your chatroom application by adding more nodes to your Phoenix cluster or using a load balancer.
- **Add more features**: Consider adding more features to your chatroom, such as private messaging, file sharing, or user authentication.
- **Explore Phoenix documentation**: Refer to the [Phoenix documentation](https://hexdocs.pm/phoenix/) for more information on building real-time applications with Elixir and Phoenix.
- **Join the Elixir community**: Engage with the Elixir community through forums, social media, and meetups to learn more about best practices and new features.
- **Contribute to open-source projects**: Consider contributing to open-source Elixir projects on GitHub to gain experience and collaborate with other developers.
- **Stay updated**: Keep an eye on the latest Elixir and Phoenix releases to take advantage of new features and improvements.
- **Learn from examples**: Explore existing open-source chatroom projects built with Elixir and Phoenix to learn from their architecture and implementation.
- **Experiment with different architectures**: Try building the chatroom using different architectures, such as microservices or serverless, to see how they affect performance and scalability.
- **Optimize performance**: Use tools like Ecto and PostgreSQL to optimize database interactions and improve the performance of your chatroom application.
- **Implement security measures**: Ensure that your chatroom application is secure by implementing measures such as input validation, authentication, and encryption.


