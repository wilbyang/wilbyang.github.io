---
layout: post
title:  "C++ if constexpr"
date:   2025-04-12 21:16:02 +0200
categories: C++
---

# 使用 `if constexpr` 替代 SFINAE 的示例

`if constexpr` 是 C++17 引入的特性，可以在编译时根据条件选择不同的代码路径，这大大简化了之前需要使用 SFINAE (Substitution Failure Is Not An Error) 技术实现的很多场景。

## 示例 1：根据类型选择不同实现

### 传统 SFINAE 方式
```cpp
template <typename T>
typename std::enable_if<std::is_integral<T>::value, void>::type
process(T value) {
    std::cout << "Processing integral: " << value << "\n";
}

template <typename T>
typename std::enable_if<std::is_floating_point<T>::value, void>::type
process(T value) {
    std::cout << "Processing floating point: " << value << "\n";
}
```

### `if constexpr` 方式
```cpp
template <typename T>
void process(T value) {
    if constexpr (std::is_integral_v<T>) {
        std::cout << "Processing integral: " << value << "\n";
    } else if constexpr (std::is_floating_point_v<T>) {
        std::cout << "Processing floating point: " << value << "\n";
    } else {
        static_assert(false, "Unsupported type");
    }
}
```

## 示例 2：检查成员函数是否存在

### 传统 SFINAE 方式
```cpp
template <typename T, typename = void>
struct has_serialize : std::false_type {};

template <typename T>
struct has_serialize<T, std::void_t<decltype(std::declval<T>().serialize())>> : std::true_type {};

template <typename T>
void save(const T& obj) {
    if (has_serialize<T>::value) {
        obj.serialize();
    } else {
        // 其他保存方式
    }
}
```

### `if constexpr` 方式
```cpp
template <typename T>
void save(const T& obj) {
    if constexpr (requires { obj.serialize(); }) {
        obj.serialize();
    } else {
        // 其他保存方式
    }
}
```

## 示例 3：根据容器类型选择不同算法

### 传统 SFINAE 方式
```cpp
template <typename Container>
auto getFirstElement(const Container& c) -> typename Container::value_type {
    return c[0];
}

template <typename Container>
auto getFirstElement(const Container& c) -> decltype(c.front()) {
    return c.front();
}
```

### `if constexpr` 方式
```cpp
template <typename Container>
auto getFirstElement(const Container& c) {
    if constexpr (requires { c[0]; }) {
        return c[0];
    } else if constexpr (requires { c.front(); }) {
        return c.front();
    } else {
        static_assert(false, "Container doesn't support [] or front()");
    }
}
```

## 示例 4：处理可选参数

### 传统 SFINAE 方式
```cpp
template <typename T>
void print(const T& value, typename std::enable_if<std::is_arithmetic<T>::value>::type* = nullptr) {
    std::cout << "Number: " << value << "\n";
}

template <typename T>
void print(const T& value, typename std::enable_if<!std::is_arithmetic<T>::value>::type* = nullptr) {
    std::cout << "Other: " << value << "\n";
}
```

### `if constexpr` 方式
```cpp
template <typename T>
void print(const T& value) {
    if constexpr (std::is_arithmetic_v<T>) {
        std::cout << "Number: " << value << "\n";
    } else {
        std::cout << "Other: " << value << "\n";
    }
}
```

## 示例 5：处理不同类型的大小

### 传统 SFINAE 方式
```cpp
template <typename T>
auto getSize(const T& obj) -> decltype(obj.size()) {
    return obj.size();
}

template <typename T>
auto getSize(const T& obj) -> decltype(sizeof(obj)) {
    return sizeof(obj);
}
```

### `if constexpr` 方式
```cpp
template <typename T>
auto getSize(const T& obj) {
    if constexpr (requires { obj.size(); }) {
        return obj.size();
    } else {
        return sizeof(obj);
    }
}
```

## 优势总结

1. 代码更简洁易读
2. 不需要编写多个重载版本
3. 编译错误信息更友好
4. 减少模板实例化数量
5. 可以在函数内部处理多种情况，而不是分散在多个重载中

注意：`if constexpr` 是编译时条件判断，不会生成未选择路径的代码，这与运行时 `if` 语句有本质区别。