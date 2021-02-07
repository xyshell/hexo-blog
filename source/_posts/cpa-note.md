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

## String methods

### compare

str1.compare(str2)

- str1.compare(str2) == 0 when str1 == str2

- str1.compare(str2) > 0 when str1 > str2

- str1.compare(str2) < 0 when str1 < str2

S.compare(substr_start, substr_length, other_string)

S.compare(substr_start, substr_length, other_string, other_substr_start, other_substr_length)

### substr

```cpp
string newstr = oldstr.substr(substring_start_position, length_of_substring)
```

### length, size, capacity, max_size

```cpp
int string_size = S.size();

int string_length = S.length();

int string_capacity = s.capacity();

int string_max_size = s.max_size();
```

```cpp
TheString.reserve(100);
```

### reserve, resize, clear, empty

```cpp
bool is_empty = TheString.empty();

TheString.resize(50,'?');

TheString.resize(4);

TheString.clear();
```

### find

```cpp
int where_it_begins = S.find(another_string, start_here);

int where_it_is = S.find(any_character, start_here);
```

```cpp
int comma = greeting.find(',');
if(comma != string::npos){
	//found
};
```

### append, push_back, insert

```cpp
NewString.append(TheString);
NewString.append(TheString,0,3);
NewString.append(2,'!');
```

```cpp
TheString.push_back(car);
```

```cpp
string quote = "Whyserious?", anyword = "monsoon";
quote.insert(3,2,' ').insert(4,anyword,3,2); // Why so serious?
```

### assign

```cpp
string sky; 
sky.assign(80,'*');
```

### replace

```cpp
string ToDo = "I'll think about that in one hour"; 
string Schedule = "today yesterday tomorrow";

ToDo.replace(22, 12, Schedule, 16, 8); // I'll think about that tomorrow
```

### erase

```cpp
string WhereAreWe = "I've got a feeling we're not in Kansas anymore"; 

WhereAreWe.erase(38, 8).erase(25, 4); // I've got a feeling we're in Kansas
TheString.erase();
```

### swap

```cpp
string Drink = "A martini";
string Needs = "Shaken, not stirred";
	
Drink.swap(Needs);
```