---
layout: post
title:  "Faking reflection with Clang builtins"
categories: Programming
tags: C++ clang hackerman
---

```c++
#include <iostream>

struct Foo {
    int primitive;
    std::string_view str;
    std::vector<int> vector;
};

int main() {
    for (auto member : reflect<Foo>()) {
        std::cout << member.type << " " << member.name << std::endl;
    }
    return 0;
}
```

```
int primitive
std::string_view str
std::vector<int> vector
```


```c++
#include <string_view>
#include <vector>

struct member {
    std::string_view type;
    std::string_view name;
};
using members = std::vector<member>;

struct helper {
    using CC = const char*;

    static void cback(members& m, CC, auto&&...) {
        // Ignore
    }
    static void cback(members& m, CC, CC, CC t, CC n, auto&&) {
        m.push_back({t,n});
    }
};

template<typename T>
members reflect() {
    members m;

    // https://clang.llvm.org/docs/LanguageExtensions.html#builtin-dump-struct
    //
    // Recent additions allow this builtin to take an overload set as the 
    // second parameter as opposed to variadic function pointer, enabling
    // all sorts of shenanigans.
    __builtin_dump_struct((T*) nullptr, helper::cback, m);
    return m;
}
```
