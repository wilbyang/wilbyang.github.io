---
layout: post
title:  "C++ Variant Visit"
date:   2025-04-12 21:16:02 +0200
categories: C++
---
## C++ Variant Visit
`std::visit` is a powerful utility in C++ that allows you to apply a visitor to the value contained in a `std::variant`. It provides a way to handle different types stored in a variant without needing to manually check the type.
### Basic Syntax
```cpp
#include <iostream>
#include <string>
#include <variant>

struct Quit {};
struct Move { int x, y; };
struct Write { std::string text; };
struct ChangeColor { int r, g, b; };

template<class... Ts>
struct overloaded : Ts... { using Ts::operator()...; };
template<class... Ts>
overloaded(Ts...) -> overloaded<Ts...>;

using Message = std::variant<Quit, Move, Write, ChangeColor>;

int main() {
    Message messages[] = {
        Quit{},
        Move{10, 20},
        Write{"Hello, World!"},
        ChangeColor{255, 0, 0}
    };

    auto visitor = overloaded{
        [](const Quit&) { std::cout << "Quit\n"; },
        [](const Move& m) { std::cout << "Move: " << m.x << ", " << m.y << "\n"; },
        [](const Write& w) { std::cout << "Write: " << w.text << "\n"; },
        [](const ChangeColor& c) { std::cout << "Color: " << c.r << ", " << c.g << ", " << c.b << "\n"; }
    };

    for (const auto& message : messages) {
        std::visit(visitor, message);
    }

    return 0;
}

```
### note

- `std::visit` requires the visitor to be exhaustive during compiling time.

- while for Rust, its enum and pattern matching system can use `non_exhaustive` attribute to allow for future extensibility.
non_exhaustive indicates that the enum may have additional variants added in the future, and the compiler will not require exhaustive matching for all possible variants.
e.g.

```rust
#[non_exhaustive]
pub enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write { text: String },
    ChangeColor { r: u8, g: u8, b: u8 },
}

fn main() {
    let messages = vec![
        Message::Quit,
        Message::Move { x: 10, y: 20 },
        Message::Write { text: String::from("Hello, World!") },
        Message::ChangeColor { r: 255, g: 0, b: 0 },
    ];

    for message in messages {
        match message {
            Message::Quit => println!("Quit"),
            Message::Move { x, y } => println!("Move: {}, {}", x, y),
            Message::Write { text } => println!("Write: {}", text),
            Message::ChangeColor { r, g, b } => println!("Color: {}, {}, {}", r, g, b),
            _ => println!("Unknown message type"), // This will handle any new variants added in the future
        }
    }
}

```
this allows you to add new variants in the future without breaking existing code.

P.S. Within the defining crate, non_exhaustive has no effect.




