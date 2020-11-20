---
title: cpa-note
date: 2020-11-20 17:19:42
tags: cpp, cpa
toc: true
---

# CPA – C++ Certified Associate Programmer

## Readings

- [The C++ Programming Language (4th Edition)](https://www.stroustrup.com/4th.html) by Bjarne Stroustrup
- Thinking in C++ by Bruce Eckel
- [Jumping into C++.](https://www.cprogramming.com/c++book/?inl=sb)
- [Effective C++.](https://www.amazon.com/gp/product/0321334876?ie=UTF8&tag=aristeia.com-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0321334876) by Scott Meyers

More books: https://stackoverflow.com/questions/388242/the-definitive-c-book-guide-and-list

## Reference

- https://isocpp.org/std
- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#main)

## cout manipulator

```c++
int byte = 255;
cout << "Byte in hex: " << hex << byte;
cout << "Byte in decimal: " << dec << byte;
cout << "Byte in octal: " << oct << byte;
```

```c++
#include <iostream>
#include <iomanip>

using namespace std;
int main(void)
{
	int byte = 255;
	cout << setbase(16) << byte;
	return 0;
}
```

```c++
float x = 2.5, y = 0.0000000025;
cout << fixed << x << " " << y << endl;
cout << scientific << x << " " << y << endl;
```