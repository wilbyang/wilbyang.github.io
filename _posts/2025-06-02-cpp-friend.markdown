---
layout: post
title: C++中的友元函数和友元类
categories: [C++]
tags: [C++, 友元函数, 友元类]
description: C++中的友元函数和友元类的概念、用法以及注意事项。
date: 2025-06-02 10:00:00 +0800
---
# C++中`friend`关键字的实用情景

`friend`是C++中一个特殊的关键字，它允许一个类或函数访问另一个类的私有(private)和受保护(protected)成员。下面是`friend`的一些实用情景：

## 1. 运算符重载

当需要重载运算符（特别是二元运算符）时，`friend`非常有用：

```cpp
class Complex {
private:
    double real, imag;
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // 声明友元函数用于运算符重载
    friend Complex operator+(const Complex& a, const Complex& b);
};

Complex operator+(const Complex& a, const Complex& b) {
    return Complex(a.real + b.real, a.imag + b.imag);
}
```

## 2. 提高性能的辅助函数

当外部函数需要频繁访问类的私有成员时，声明为`friend`可以避免使用getter/setter带来的性能开销：

```cpp
class Image {
private:
    unsigned char* pixels;
    int width, height;
public:
    friend void applyFilter(Image& img, const Filter& filter);
};

void applyFilter(Image& img, const Filter& filter) {
    // 直接访问私有成员，无需通过公有接口
    for (int i = 0; i < img.width * img.height; ++i) {
        // 处理像素
    }
}
```

## 3. 单元测试

在单元测试中，测试类可能需要访问被测类的私有成员：

```cpp
class BankAccount {
private:
    double balance;
public:
    friend class BankAccountTest; // 测试类可以访问私有成员
};

class BankAccountTest {
public:
    static void test() {
        BankAccount acc;
        acc.balance = 100.0; // 直接访问私有成员进行测试
        // 进行测试...
    }
};
```

## 4. 工厂模式

当使用工厂模式时，工厂类可能需要访问产品类的私有构造函数：

```cpp
class Product {
private:
    Product() {} // 私有构造函数
    friend class ProductFactory;
};

class ProductFactory {
public:
    static Product* create() {
        return new Product(); // 可以访问私有构造函数
    }
};
```

## 5. 数据序列化/反序列化

序列化函数需要访问类的私有成员：

```cpp
class UserData {
private:
    std::string name;
    int age;
    std::vector<std::string> preferences;
public:
    friend void serialize(const UserData& data, std::ostream& out);
    friend void deserialize(UserData& data, std::istream& in);
};
```

## 注意事项

1. `friend`破坏了封装性，应谨慎使用
2. 只在确实需要时才使用`friend`
3. 尽量保持`friend`关系的单向性
4. 考虑是否有替代方案（如公有接口）

`friend`是C++提供的一种灵活机制，合理使用可以在不破坏封装性的前提下提供必要的访问权限。