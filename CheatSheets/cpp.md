# C++ Complete Reference Guide
> From Scratch to Expert Level — Everything You Need to Master C++

---

## 1. Environment Setup & Compilation

### Installing a Compiler

**Linux (GCC)**
```bash
sudo apt install g++         # Debian/Ubuntu
sudo dnf install gcc-c++     # Fedora/RHEL
```

**macOS (Clang via Xcode)**
```bash
xcode-select --install
```

**Windows**
- Install [MinGW-w64](https://www.mingw-w64.org/) or use MSVC via Visual Studio
- Or use WSL (Windows Subsystem for Linux)

### Compiling & Running

```bash
g++ hello.cpp -o hello           # Basic compile
./hello                          # Run

g++ -std=c++17 main.cpp -o app   # Specify C++ standard
g++ -Wall -Wextra -O2 main.cpp   # Enable warnings + optimization
g++ -g main.cpp -o app           # Debug symbols (for GDB)
```

### Common Compiler Flags

| Flag | Meaning |
|------|---------|
| `-std=c++17` | Use C++17 standard |
| `-Wall` | Enable common warnings |
| `-Wextra` | Enable extra warnings |
| `-O0` | No optimization (default) |
| `-O2` | Standard optimization |
| `-O3` | Aggressive optimization |
| `-g` | Debug info for GDB |
| `-DDEBUG` | Define a preprocessor macro |

### CMake (Build System)

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.15)
project(MyApp)

set(CMAKE_CXX_STANDARD 17)

add_executable(MyApp main.cpp utils.cpp)
```

```bash
mkdir build && cd build
cmake ..
make
```

---

## 2. Program Structure & Basics

### Hello World

```cpp
#include <iostream>
using namespace std;

int main() {
    cout << "Hello, World!" << endl;
    return 0;
}
```

### Preprocessor Directives

```cpp
#include <iostream>        // Standard library header
#include "myfile.h"        // Local header file
#define PI 3.14159         // Macro constant
#define MAX(a,b) ((a)>(b)?(a):(b))  // Macro function (prefer inline instead)
#undef PI                  // Remove macro

#ifdef DEBUG
    cout << "Debug mode" << endl;
#endif

#pragma once               // Header guard (modern alternative to ifndef)
```

### Header Guards

```cpp
// myheader.h
#ifndef MYHEADER_H
#define MYHEADER_H

// declarations here

#endif // MYHEADER_H
```

### Namespaces

```cpp
namespace Math {
    double PI = 3.14159;

    int add(int a, int b) {
        return a + b;
    }
}

// Usage
double x = Math::PI;
int sum = Math::add(3, 4);

// Bring into scope
using namespace Math;
using Math::add;  // Bring only one symbol

// Nested namespaces (C++17)
namespace Outer::Inner {
    void func() {}
}
```

### Input / Output

```cpp
#include <iostream>
#include <iomanip>    // For formatting

// Output
cout << "Value: " << x << endl;
cout << "Value: " << x << "\n";   // "\n" is faster than endl (no flush)
cerr << "Error!" << endl;         // stderr

// Input
int n;
cin >> n;

string name;
cin >> name;               // Reads until whitespace
getline(cin, name);        // Reads full line

// Formatting output
cout << fixed << setprecision(2) << 3.14159;  // 3.14
cout << setw(10) << "hello";                  // Right-aligned in 10 chars
cout << left << setw(10) << "hello";          // Left-aligned
cout << hex << 255;                           // ff
cout << oct << 8;                             // 10
cout << boolalpha << true;                    // true
```

---

## 3. Data Types & Variables

### Fundamental Types

| Type | Size | Range / Notes |
|------|------|---------------|
| `bool` | 1 byte | `true` / `false` |
| `char` | 1 byte | -128 to 127 |
| `unsigned char` | 1 byte | 0 to 255 |
| `short` | 2 bytes | -32,768 to 32,767 |
| `int` | 4 bytes | ~±2.1 billion |
| `long` | 4–8 bytes | Platform-dependent |
| `long long` | 8 bytes | ~±9.2 × 10¹⁸ |
| `float` | 4 bytes | ~7 decimal digits |
| `double` | 8 bytes | ~15 decimal digits |
| `long double` | 16 bytes | ~18–19 decimal digits |
| `void` | — | No value |

### Fixed-Width Integer Types (C++11)

```cpp
#include <cstdint>

int8_t   a = 127;
int16_t  b = 32000;
int32_t  c = 2000000000;
int64_t  d = 9000000000LL;
uint32_t e = 4000000000U;
```

### Variable Declaration & Initialization

```cpp
int x;           // Declared (undefined value — dangerous!)
int y = 10;      // Copy initialization
int z(10);       // Direct initialization
int w{10};       // Uniform initialization (C++11) — prevents narrowing
auto val = 42;   // Type deduced as int

// Constants
const int MAX = 100;
constexpr int SIZE = 256;  // Evaluated at compile time

// Multiple declarations
int a = 1, b = 2, c = 3;
```

### Type Aliases

```cpp
typedef unsigned long ulong;   // Old style
using ulong = unsigned long;   // Modern (C++11)

using Matrix = vector<vector<int>>;
```

### Type Conversion

```cpp
// Implicit
double d = 5;       // int → double

// Explicit (C-style — avoid)
int x = (int)3.9;

// C++ style casts (prefer these)
int a = static_cast<int>(3.9);         // Safe compile-time cast
double* dp = reinterpret_cast<double*>(&a);  // Reinterpret bits (dangerous)
const int ci = 10;
int* p = const_cast<int*>(&ci);        // Remove const (dangerous)
Base* b = dynamic_cast<Derived*>(ptr); // Safe polymorphic downcast

// Checking types
#include <typeinfo>
cout << typeid(x).name();
```

### `auto` and `decltype`

```cpp
auto x = 42;             // int
auto y = 3.14;           // double
auto z = "hello";        // const char*
auto& ref = x;           // int&

decltype(x) a = 10;      // Same type as x
decltype(x + y) b = 5.0; // double
```

---

## 4. Operators

### Arithmetic

```cpp
int a = 10, b = 3;

a + b    // 13
a - b    // 7
a * b    // 30
a / b    // 3  (integer division)
a % b    // 1  (modulo)
++a      // pre-increment: a becomes 11, returns 11
a++      // post-increment: returns 10, then a becomes 11
--b      // pre-decrement
```

### Assignment

```cpp
a = 5;
a += 5;   // a = a + 5
a -= 5;
a *= 2;
a /= 2;
a %= 3;
a &= 0xFF;
a |= 0x01;
a ^= 0x10;
a <<= 1;  // left shift
a >>= 1;  // right shift
```

### Comparison & Logical

```cpp
a == b   // equal
a != b   // not equal
a > b    // greater than
a < b    // less than
a >= b   // greater or equal
a <= b   // less or equal

a && b   // logical AND
a || b   // logical OR
!a       // logical NOT
```

### Bitwise

```cpp
a & b    // AND
a | b    // OR
a ^ b    // XOR
~a       // NOT (bitwise complement)
a << 2   // left shift by 2 (multiply by 4)
a >> 1   // right shift by 1 (divide by 2)
```

### Other Operators

```cpp
// Ternary
int max = (a > b) ? a : b;

// Comma (evaluates both, returns last)
int x = (a++, b++, a + b);

// sizeof
sizeof(int);     // 4
sizeof(arr);     // Total array size in bytes

// Pointer operators
int* p = &a;     // Address-of
*p = 10;         // Dereference

// Scope resolution
std::cout << ...;
ClassName::staticMember;

// Spaceship operator (C++20)
auto result = a <=> b;  // Returns strong/weak/partial ordering
```

### Operator Precedence (High → Low)

| Precedence | Operators |
|-----------|-----------|
| Highest | `::` |
| | `()` `[]` `->` `.` `++` `--` (postfix) |
| | `++` `--` (prefix) `~` `!` `-` `+` `*` `&` `sizeof` `new` `delete` |
| | `.*` `->*` |
| | `*` `/` `%` |
| | `+` `-` |
| | `<<` `>>` |
| | `<=>` |
| | `<` `<=` `>` `>=` |
| | `==` `!=` |
| | `&` |
| | `^` |
| | `\|` |
| | `&&` |
| | `\|\|` |
| | `?:` |
| | `=` `+=` `-=` etc. |
| Lowest | `,` |

---

## 5. Control Flow

### if / else

```cpp
if (x > 0) {
    cout << "positive";
} else if (x < 0) {
    cout << "negative";
} else {
    cout << "zero";
}

// if with initializer (C++17)
if (int n = compute(); n > 0) {
    cout << "positive: " << n;
}
```

### switch

```cpp
switch (day) {
    case 1:
        cout << "Monday";
        break;
    case 2:
        cout << "Tuesday";
        break;
    case 6:
    case 7:
        cout << "Weekend";
        break;
    default:
        cout << "Other";
}

// switch with initializer (C++17)
switch (int x = getValue(); x) {
    case 0: break;
    default: break;
}
```

### Loops

```cpp
// for
for (int i = 0; i < 10; i++) {
    if (i == 5) continue;   // skip to next iteration
    if (i == 8) break;      // exit loop
    cout << i;
}

// while
int i = 0;
while (i < 10) {
    cout << i++;
}

// do-while (executes at least once)
do {
    cout << i++;
} while (i < 10);

// Range-based for (C++11)
vector<int> v = {1, 2, 3, 4, 5};
for (int x : v) cout << x;
for (auto& x : v) x *= 2;   // Modify elements with reference
for (const auto& x : v) cout << x;  // Read-only reference (prefer for large objects)

// Nested loop label workaround (use goto sparingly)
for (int i = 0; i < 5; i++)
    for (int j = 0; j < 5; j++)
        if (i == j) goto done;
done:
```

### goto (rarely needed, avoid)

```cpp
int x = 0;
start:
    if (x < 5) {
        x++;
        goto start;
    }
```

---

## 6. Functions

### Basics

```cpp
// Declaration (prototype)
int add(int a, int b);

// Definition
int add(int a, int b) {
    return a + b;
}

// Void function
void printLine() {
    cout << "---" << endl;
}

// Multiple return values via struct or pair
pair<int, int> minMax(vector<int>& v) {
    return {*min_element(v.begin(), v.end()),
            *max_element(v.begin(), v.end())};
}
auto [mn, mx] = minMax(v);  // C++17 structured bindings
```

### Parameter Passing

```cpp
// By value (copy) — modifications don't affect original
void increment(int x) { x++; }

// By reference — modifications affect original
void increment(int& x) { x++; }

// By const reference — no copy, no modification (ideal for large objects)
void print(const string& s) { cout << s; }

// By pointer
void increment(int* x) { (*x)++; }
increment(&n);

// By move (C++11) — transfer ownership
void consume(string&& s) { ... }

// Output parameters
void getValues(int& out1, double& out2) {
    out1 = 42;
    out2 = 3.14;
}
```

### Default Parameters

```cpp
double power(double base, int exp = 2) {
    return pow(base, exp);
}

power(3);      // 9.0
power(3, 3);   // 27.0
```

### Function Overloading

```cpp
int add(int a, int b) { return a + b; }
double add(double a, double b) { return a + b; }
string add(string a, string b) { return a + b; }
// Compiler picks based on argument types
```

### Inline Functions

```cpp
inline int square(int x) { return x * x; }
// Suggests compiler to inline the code (avoid function call overhead)
// Modern compilers decide themselves, but inline is still needed for headers
```

### Recursive Functions

```cpp
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

// Tail recursion (some compilers optimize)
int factTail(int n, int acc = 1) {
    if (n <= 1) return acc;
    return factTail(n - 1, n * acc);
}

// Fibonacci with memoization
unordered_map<int, long long> memo;
long long fib(int n) {
    if (n <= 1) return n;
    if (memo.count(n)) return memo[n];
    return memo[n] = fib(n-1) + fib(n-2);
}
```

### Lambda Functions

```cpp
// Basic lambda
auto greet = []() { cout << "Hello"; };
greet();

// With parameters and return type
auto add = [](int a, int b) -> int { return a + b; };

// Capture by value
int x = 10;
auto addX = [x](int y) { return x + y; };

// Capture by reference
auto increment = [&x]() { x++; };

// Capture all by value
auto f1 = [=]() { return x + 1; };

// Capture all by reference
auto f2 = [&]() { x++; };

// Mutable lambda (modify captured copy)
auto counter = [count = 0]() mutable { return ++count; };

// Generic lambda (C++14)
auto multiply = [](auto a, auto b) { return a * b; };

// Immediately invoked lambda
int result = [](int a, int b) { return a + b; }(3, 4);

// Store lambda in std::function
#include <functional>
function<int(int, int)> op = [](int a, int b) { return a + b; };
```

### Function Pointers

```cpp
// Declare a function pointer type
int (*operation)(int, int);

// Assign
operation = add;

// Call
int result = operation(3, 4);

// As parameter
void apply(int a, int b, int (*func)(int, int)) {
    cout << func(a, b);
}

// Using typedef/using
typedef int (*BinaryOp)(int, int);
using BinaryOp = int(*)(int, int);
```

### `std::function` (more flexible)

```cpp
#include <functional>

function<int(int, int)> f;
f = add;                                // Regular function
f = [](int a, int b){ return a + b; }; // Lambda
f = multiplier;                         // Functor
```

---

## 7. Arrays & Strings

### C-Style Arrays

```cpp
int arr[5];                    // Uninitialized
int nums[5] = {1, 2, 3, 4, 5};
int vals[] = {10, 20, 30};     // Size inferred as 3
int zeros[10] = {};            // All zeroes

// Access
nums[0] = 99;
int x = nums[2];

// Size
int size = sizeof(nums) / sizeof(nums[0]);   // 5

// Multi-dimensional
int matrix[3][4];
int grid[2][3] = {{1,2,3}, {4,5,6}};
cout << grid[1][2];  // 6

// Passing to functions (decays to pointer)
void print(int arr[], int size) { ... }
void print(int arr[5]) { ... }    // Size hint is ignored
void print(int (*arr)[4], int rows) { ... }  // For 2D arrays
```

### `std::array` (C++11 — prefer over C-arrays)

```cpp
#include <array>

array<int, 5> arr = {1, 2, 3, 4, 5};
arr[0] = 10;
arr.at(1);        // Bounds-checked access
arr.size();       // 5
arr.front();      // First element
arr.back();       // Last element
arr.fill(0);      // Set all to 0

// Works with algorithms
sort(arr.begin(), arr.end());
```

### C-Style Strings

```cpp
char str[] = "Hello";           // char[6] including '\0'
char str2[10] = "Hello";

#include <cstring>
strlen(str);                    // 5 (excludes '\0')
strcpy(dest, src);              // Copy (dangerous — prefer strncpy)
strncpy(dest, src, n);
strcat(dest, src);              // Append (dangerous)
strcmp(a, b);                   // 0 if equal, <0, >0 otherwise
```

### `std::string` (prefer this)

```cpp
#include <string>

string s = "Hello";
string s2("World");
string s3(5, 'x');       // "xxxxx"
string s4 = s + " " + s2;

// Access
s[0];              // 'H'
s.at(0);           // 'H' with bounds checking
s.front();         // 'H'
s.back();          // 'o'

// Size & capacity
s.length();        // Same as size()
s.size();
s.empty();
s.capacity();
s.reserve(100);    // Pre-allocate

// Modification
s.append(" World");
s += " World";
s.insert(5, " Awesome");   // Insert at index
s.erase(5, 7);             // Erase 7 chars at index 5
s.replace(0, 5, "Hi");     // Replace chars

// Search
s.find("lo");              // Returns index or string::npos
s.rfind("l");              // Find from end
s.find_first_of("aeiou");  // Find first vowel
s.find_last_of("aeiou");

// Substrings
s.substr(0, 5);   // First 5 chars
s.substr(6);      // From index 6 to end

// Comparisons
s == "Hello";
s.compare("Hello");   // 0 if equal

// Conversion
stoi("42");       // string to int
stod("3.14");     // string to double
stof("3.14");     // string to float
stoll("123456");  // string to long long
to_string(42);    // int to string

// Iterating
for (char c : s) cout << c;
for (auto it = s.begin(); it != s.end(); ++it) cout << *it;
```

### `std::string_view` (C++17 — non-owning)

```cpp
#include <string_view>

string_view sv = "Hello, World";  // No copy
string_view sv2 = s;              // View into existing string
sv.substr(0, 5);                  // Also no copy
sv.starts_with("Hello");          // C++20
sv.ends_with("World");            // C++20
```

---

## 8. Pointers & References

### Pointers

```cpp
int x = 42;
int* ptr = &x;     // ptr holds address of x

*ptr;              // 42 (dereference)
*ptr = 100;        // Changes x to 100
ptr;               // Memory address

// Null pointer
int* p = nullptr;  // C++11 (prefer over NULL or 0)
if (p != nullptr) { ... }

// Pointer arithmetic
int arr[5] = {1, 2, 3, 4, 5};
int* p = arr;      // Points to arr[0]
p++;               // Points to arr[1]
*(p + 2);          // arr[3] value
p - arr;           // Distance: 1

// Pointers to pointers
int** pp = &ptr;
**pp = 200;        // Changes x to 200

// const and pointers
const int* p1 = &x;     // Pointer to const — can't modify *p1
int* const p2 = &x;     // Const pointer — can't change where p2 points
const int* const p3 = &x; // Both const

// Void pointer (type-erased)
void* vp = &x;
int* ip = static_cast<int*>(vp);
```

### References

```cpp
int x = 42;
int& ref = x;      // ref is an alias for x

ref = 100;         // x is now 100
ref == x;          // always true

// Must be initialized, cannot be reseated (unlike pointers)

// const reference
const int& cref = x;   // Can read but not modify
const int& temp = 42;  // Can bind to rvalue (temporary)

// References as parameters (avoids copy)
void swap(int& a, int& b) {
    int tmp = a;
    a = b;
    b = tmp;
}

// Returning reference
int& getElement(vector<int>& v, int i) {
    return v[i];  // Allows: getElement(v, 2) = 99;
}
// Warning: never return reference to local variable!
```

### Pointer vs Reference Summary

| | Pointer | Reference |
|--|---------|-----------|
| Null value | Can be `nullptr` | Cannot be null |
| Reassignable | Yes | No (fixed after init) |
| Arithmetic | Yes | No |
| Must init | No | Yes |
| Syntax to access | `*ptr` / `ptr->` | `ref` directly |
| Use when | Optional/array/null possible | Alias/must exist |

---

## 9. Dynamic Memory Management

### `new` and `delete`

```cpp
// Single object
int* p = new int;        // Allocate
*p = 42;
delete p;                // Deallocate
p = nullptr;             // Good practice

// With initialization
int* p2 = new int(42);
delete p2;

// Array
int* arr = new int[10];
arr[0] = 1;
delete[] arr;            // Must use delete[]!

// 2D array
int** matrix = new int*[rows];
for (int i = 0; i < rows; i++)
    matrix[i] = new int[cols];
// Cleanup:
for (int i = 0; i < rows; i++)
    delete[] matrix[i];
delete[] matrix;
```

### Smart Pointers (C++11 — prefer over raw owning pointers)

```cpp
#include <memory>

// unique_ptr — exclusive ownership
unique_ptr<int> p = make_unique<int>(42);   // C++14
*p = 100;
p.reset();            // Delete the object
p.release();          // Give up ownership (returns raw pointer)
unique_ptr<int> p2 = move(p);  // Transfer ownership

// unique_ptr for arrays
unique_ptr<int[]> arr = make_unique<int[]>(10);
arr[0] = 1;

// shared_ptr — shared ownership (reference counted)
shared_ptr<int> sp1 = make_shared<int>(42);
shared_ptr<int> sp2 = sp1;   // Reference count = 2
sp1.use_count();              // 2
sp1.reset();                  // Count = 1
// Deleted when last shared_ptr is destroyed

// weak_ptr — non-owning observer (breaks circular refs)
weak_ptr<int> wp = sp2;
if (auto sp = wp.lock()) {   // Get shared_ptr if still alive
    cout << *sp;
}
wp.expired();                 // Check if object was deleted
```

### Custom Deleters

```cpp
// unique_ptr with custom deleter
auto fileDeleter = [](FILE* f) { fclose(f); };
unique_ptr<FILE, decltype(fileDeleter)> fp(fopen("file.txt", "r"), fileDeleter);
```

### RAII (Resource Acquisition Is Initialization)

```cpp
// Wrap resources in classes — automatically cleaned up on scope exit
class FileWrapper {
    FILE* file;
public:
    FileWrapper(const char* name, const char* mode)
        : file(fopen(name, mode)) {
        if (!file) throw runtime_error("Cannot open file");
    }
    ~FileWrapper() {
        if (file) fclose(file);
    }
    FILE* get() { return file; }
    // Disable copy
    FileWrapper(const FileWrapper&) = delete;
    FileWrapper& operator=(const FileWrapper&) = delete;
};
```

---

## 10. Object-Oriented Programming

### Classes & Structs

```cpp
class Dog {
private:
    string name;
    int age;

public:
    // Default constructor
    Dog() : name("Unknown"), age(0) {}

    // Parameterized constructor
    Dog(string n, int a) : name(n), age(a) {}

    // Copy constructor
    Dog(const Dog& other) : name(other.name), age(other.age) {}

    // Destructor
    ~Dog() {
        cout << name << " destroyed" << endl;
    }

    // Getters / Setters
    string getName() const { return name; }     // const: doesn't modify object
    void setName(const string& n) { name = n; }

    // Member function
    void bark() const {
        cout << name << " says: Woof!" << endl;
    }

    // Static member
    static int count;
    static int getCount() { return count; }
};

int Dog::count = 0;  // Define static member outside class

// Struct (same as class but default public)
struct Point {
    double x, y;
    double distance() const {
        return sqrt(x*x + y*y);
    }
};
```

### Constructor Types

```cpp
class Widget {
    int val;
    string name;
    vector<int> data;

public:
    // Default constructor
    Widget() = default;

    // Delegating constructor (C++11)
    Widget(int v) : Widget(v, "default") {}
    Widget(int v, string n) : val(v), name(n) {}

    // Copy constructor
    Widget(const Widget&) = default;    // Compiler-generated
    Widget(const Widget& other) : val(other.val), name(other.name), data(other.data) {}

    // Move constructor (C++11)
    Widget(Widget&& other) noexcept
        : val(other.val), name(move(other.name)), data(move(other.data)) {
        other.val = 0;
    }

    // Copy assignment
    Widget& operator=(const Widget&) = default;

    // Move assignment
    Widget& operator=(Widget&& other) noexcept {
        if (this != &other) {
            val = other.val;
            name = move(other.name);
            data = move(other.data);
            other.val = 0;
        }
        return *this;
    }

    // Destructor
    ~Widget() = default;

    // Deleted functions
    Widget(const Widget&) = delete;   // Prevents copying
};
```

### Inheritance

```cpp
class Animal {
protected:
    string name;
    int age;

public:
    Animal(string n, int a) : name(n), age(a) {}

    virtual void speak() const {
        cout << name << " makes a sound" << endl;
    }

    virtual ~Animal() {}    // Always virtual destructor in base class!

    string getName() const { return name; }
};

class Cat : public Animal {
    string color;

public:
    Cat(string n, int a, string c)
        : Animal(n, a), color(c) {}     // Call base constructor

    void speak() const override {       // override keyword catches typos
        cout << name << " says: Meow!" << endl;
    }

    void describe() const {
        cout << color << " cat named " << name << endl;
    }
};

// Multiple inheritance
class Swimmer { public: virtual void swim() {} };
class Runner  { public: virtual void run() {}  };

class Triathlete : public Swimmer, public Runner {
public:
    void swim() override { cout << "Swimming" << endl; }
    void run() override  { cout << "Running" << endl;  }
};

// Access specifiers in inheritance
class A : public Base {};     // public    → stays as-is
class B : protected Base {};  // public    → becomes protected
class C : private Base {};    // public/protected → become private
```

### Polymorphism & Virtual Functions

```cpp
class Shape {
public:
    virtual double area() const = 0;    // Pure virtual — makes class abstract
    virtual double perimeter() const = 0;
    virtual void print() const {
        cout << "Area: " << area() << endl;
    }
    virtual ~Shape() {}
};

class Circle : public Shape {
    double r;
public:
    Circle(double radius) : r(radius) {}
    double area() const override { return 3.14159 * r * r; }
    double perimeter() const override { return 2 * 3.14159 * r; }
};

class Rectangle : public Shape {
    double w, h;
public:
    Rectangle(double w, double h) : w(w), h(h) {}
    double area() const override { return w * h; }
    double perimeter() const override { return 2*(w+h); }
};

// Polymorphic usage
vector<unique_ptr<Shape>> shapes;
shapes.push_back(make_unique<Circle>(5));
shapes.push_back(make_unique<Rectangle>(4, 6));

for (auto& s : shapes) {
    s->print();  // Correct virtual dispatch
}
```

### Virtual Function Table (vtable)

Each class with virtual functions has a hidden vtable — a table of function pointers. Each object has a vptr pointing to its class's vtable. `dynamic_cast` and runtime polymorphism use this.

### `final` and `override`

```cpp
class Base {
    virtual void foo() {}
};

class Derived final : public Base {      // Cannot inherit from Derived
    void foo() override final {}         // Cannot override further
};
```

### Static Members & Functions

```cpp
class Counter {
    static int count;    // Shared across all instances
    int id;

public:
    Counter() : id(++count) {}
    static int getCount() { return count; }  // No 'this' pointer
    int getId() const { return id; }
};

int Counter::count = 0;  // Must define outside class

Counter a, b, c;
cout << Counter::getCount();  // 3
```

### Friend Functions & Classes

```cpp
class Matrix {
    int data[3][3];

    // Friend function can access private members
    friend ostream& operator<<(ostream& os, const Matrix& m);
    friend class MatrixHelper;
};

ostream& operator<<(ostream& os, const Matrix& m) {
    // Access m.data directly
    return os;
}
```

---

## 11. Operator Overloading

```cpp
class Vector2D {
public:
    double x, y;

    Vector2D(double x = 0, double y = 0) : x(x), y(y) {}

    // Arithmetic operators
    Vector2D operator+(const Vector2D& other) const {
        return {x + other.x, y + other.y};
    }

    Vector2D operator-(const Vector2D& other) const {
        return {x - other.x, y - other.y};
    }

    Vector2D operator*(double scalar) const {
        return {x * scalar, y * scalar};
    }

    // Compound assignment
    Vector2D& operator+=(const Vector2D& other) {
        x += other.x; y += other.y;
        return *this;
    }

    // Comparison
    bool operator==(const Vector2D& other) const {
        return x == other.x && y == other.y;
    }

    bool operator!=(const Vector2D& other) const {
        return !(*this == other);
    }

    // Unary
    Vector2D operator-() const { return {-x, -y}; }

    // Subscript
    double& operator[](int i) {
        return i == 0 ? x : y;
    }

    // Increment (prefix)
    Vector2D& operator++() {
        x++; y++;
        return *this;
    }

    // Increment (postfix)
    Vector2D operator++(int) {
        Vector2D tmp = *this;
        ++(*this);
        return tmp;
    }

    // Call operator (functor)
    double operator()() const {
        return sqrt(x*x + y*y);  // magnitude
    }

    // Stream operators (as friends)
    friend ostream& operator<<(ostream& os, const Vector2D& v) {
        return os << "(" << v.x << ", " << v.y << ")";
    }

    friend istream& operator>>(istream& is, Vector2D& v) {
        return is >> v.x >> v.y;
    }
};

// Non-member scalar multiplication
Vector2D operator*(double scalar, const Vector2D& v) {
    return v * scalar;
}
```

---

## 12. Templates & Generic Programming

### Function Templates

```cpp
template <typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}

max(3, 5);        // int
max(3.0, 5.0);    // double
max<double>(3, 5.0);  // Explicit instantiation

// Multiple type parameters
template <typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}

// Non-type template parameters
template <typename T, int N>
T sumArray(T (&arr)[N]) {
    T total = 0;
    for (int i = 0; i < N; i++) total += arr[i];
    return total;
}

// Variadic templates (C++11)
template <typename T>
T sum(T t) { return t; }

template <typename T, typename... Args>
T sum(T first, Args... rest) {
    return first + sum(rest...);
}

sum(1, 2, 3, 4, 5);  // 15
```

### Class Templates

```cpp
template <typename T>
class Stack {
    vector<T> data;

public:
    void push(const T& val) { data.push_back(val); }
    void push(T&& val) { data.push_back(move(val)); }

    T pop() {
        if (empty()) throw runtime_error("Stack is empty");
        T top = move(data.back());
        data.pop_back();
        return top;
    }

    T& peek() { return data.back(); }
    bool empty() const { return data.empty(); }
    size_t size() const { return data.size(); }
};

Stack<int> intStack;
Stack<string> strStack;
```

### Template Specialization

```cpp
// Primary template
template <typename T>
class TypeInfo {
public:
    static const char* name() { return "unknown"; }
};

// Full specialization
template <>
class TypeInfo<int> {
public:
    static const char* name() { return "int"; }
};

template <>
class TypeInfo<double> {
public:
    static const char* name() { return "double"; }
};

// Partial specialization
template <typename T>
class TypeInfo<T*> {
public:
    static const char* name() { return "pointer"; }
};
```

### Concepts (C++20)

```cpp
#include <concepts>

// Define a concept
template <typename T>
concept Numeric = is_integral_v<T> || is_floating_point_v<T>;

template <typename T>
concept Sortable = requires(T a, T b) {
    { a < b } -> convertible_to<bool>;
};

// Use concepts to constrain templates
template <Numeric T>
T square(T x) { return x * x; }

template <typename T>
    requires Sortable<T>
T myMax(T a, T b) { return (a < b) ? b : a; }
```

---

## 13. Standard Template Library (STL)

### Containers Overview

| Container | Header | Ordered | Duplicates | Access |
|-----------|--------|---------|------------|--------|
| `vector` | `<vector>` | Yes (insertion) | Yes | O(1) random |
| `deque` | `<deque>` | Yes | Yes | O(1) random |
| `list` | `<list>` | Yes | Yes | O(n) sequential |
| `forward_list` | `<forward_list>` | Yes | Yes | O(n) sequential |
| `array` | `<array>` | Yes | Yes | O(1) random |
| `set` | `<set>` | Sorted | No | O(log n) |
| `multiset` | `<set>` | Sorted | Yes | O(log n) |
| `map` | `<map>` | Sorted by key | No (keys) | O(log n) |
| `multimap` | `<map>` | Sorted | Yes (keys) | O(log n) |
| `unordered_set` | `<unordered_set>` | No | No | O(1) avg |
| `unordered_map` | `<unordered_map>` | No | No (keys) | O(1) avg |
| `stack` | `<stack>` | LIFO | Yes | Top only |
| `queue` | `<queue>` | FIFO | Yes | Front/back |
| `priority_queue` | `<queue>` | Heap | Yes | Top only |

### vector

```cpp
#include <vector>

vector<int> v;
vector<int> v2(5, 0);          // {0, 0, 0, 0, 0}
vector<int> v3 = {1, 2, 3};

// Adding elements
v.push_back(10);
v.emplace_back(10);             // Construct in-place (faster)
v.insert(v.begin() + 2, 99);   // Insert at index 2

// Removing elements
v.pop_back();
v.erase(v.begin() + 2);        // Remove at index 2
v.erase(v.begin(), v.begin()+3); // Remove range
v.clear();

// Access
v[0];              // No bounds check
v.at(0);           // Bounds-checked
v.front();
v.back();
v.data();          // Raw pointer to data

// Size
v.size();
v.capacity();
v.empty();
v.resize(10);      // Change size (fill with 0)
v.reserve(100);    // Reserve capacity without resizing
v.shrink_to_fit(); // Release excess memory

// Iterating
for (auto& x : v) cout << x;
for (auto it = v.begin(); it != v.end(); ++it) cout << *it;
for (auto it = v.rbegin(); it != v.rend(); ++it) cout << *it;  // Reverse
```

### map / unordered_map

```cpp
#include <map>
#include <unordered_map>

map<string, int> scores;
scores["Alice"] = 95;
scores["Bob"] = 87;
scores.insert({"Charlie", 78});
scores.emplace("Dave", 91);

// Access
scores["Alice"];                   // Creates entry if doesn't exist
scores.at("Alice");                // Throws if doesn't exist

// Check existence
scores.count("Alice");             // 0 or 1
scores.find("Bob") != scores.end();// More explicit

// Remove
scores.erase("Bob");

// Iterate (sorted by key for map)
for (auto& [name, score] : scores) {  // C++17 structured bindings
    cout << name << ": " << score << endl;
}

// unordered_map (faster O(1) avg but no order)
unordered_map<string, int> umap;
umap.reserve(100);                 // Performance hint
umap.max_load_factor(0.25);        // Reduce collisions
```

### set / unordered_set

```cpp
#include <set>

set<int> s = {5, 3, 1, 4, 2};   // Sorted: {1,2,3,4,5}
s.insert(6);
s.erase(3);
s.count(4);                     // 0 or 1
s.find(4) != s.end();

// Range operations
s.lower_bound(3);               // Iterator to first >= 3
s.upper_bound(3);               // Iterator to first > 3

for (int x : s) cout << x;     // Sorted order
```

### stack, queue, priority_queue

```cpp
#include <stack>
stack<int> st;
st.push(1);
st.top();     // Access top
st.pop();     // Remove top
st.empty();

#include <queue>
queue<int> q;
q.push(1);
q.front();    // Access front
q.back();     // Access back
q.pop();      // Remove front

// Priority queue (max-heap by default)
priority_queue<int> pq;
pq.push(5); pq.push(1); pq.push(10);
pq.top();   // 10

// Min-heap
priority_queue<int, vector<int>, greater<int>> minPQ;
```

### Algorithms

```cpp
#include <algorithm>
#include <numeric>

vector<int> v = {5, 3, 1, 4, 2};

// Sorting
sort(v.begin(), v.end());                   // Ascending
sort(v.begin(), v.end(), greater<int>());   // Descending
sort(v.begin(), v.end(), [](int a, int b){ return a > b; });
stable_sort(v.begin(), v.end());            // Preserves equal element order

// Searching
binary_search(v.begin(), v.end(), 3);       // true/false (must be sorted)
find(v.begin(), v.end(), 3);               // Iterator
find_if(v.begin(), v.end(), [](int x){ return x > 3; });
lower_bound(v.begin(), v.end(), 3);        // First >= 3
upper_bound(v.begin(), v.end(), 3);        // First > 3

// Min/max
*min_element(v.begin(), v.end());
*max_element(v.begin(), v.end());
auto [mn, mx] = minmax_element(v.begin(), v.end());

// Count & check
count(v.begin(), v.end(), 3);
count_if(v.begin(), v.end(), [](int x){ return x > 3; });
all_of(v.begin(), v.end(), [](int x){ return x > 0; });
any_of(v.begin(), v.end(), [](int x){ return x > 4; });
none_of(v.begin(), v.end(), [](int x){ return x > 10; });

// Transform
transform(v.begin(), v.end(), v.begin(), [](int x){ return x * 2; });

// Copy & fill
copy(v.begin(), v.end(), dest.begin());
fill(v.begin(), v.end(), 0);
fill_n(v.begin(), 3, -1);
iota(v.begin(), v.end(), 1);   // Fill with 1,2,3,4,...

// Remove (does NOT resize — use with erase)
auto it = remove(v.begin(), v.end(), 3);
v.erase(it, v.end());   // "Erase-remove idiom"

// Erase-remove with lambda (C++17+)
remove_if(v.begin(), v.end(), [](int x){ return x % 2 == 0; });

// Accumulate / reduce
#include <numeric>
int total = accumulate(v.begin(), v.end(), 0);
int product = accumulate(v.begin(), v.end(), 1, multiplies<int>());

// Reverse & rotate
reverse(v.begin(), v.end());
rotate(v.begin(), v.begin() + 2, v.end());  // Rotate left by 2

// Unique (removes consecutive duplicates, must sort first)
v.erase(unique(v.begin(), v.end()), v.end());

// Merge two sorted ranges
vector<int> result;
merge(a.begin(), a.end(), b.begin(), b.end(), back_inserter(result));

// Set operations (on sorted ranges)
set_union(a.begin(), a.end(), b.begin(), b.end(), back_inserter(result));
set_intersection(a.begin(), a.end(), b.begin(), b.end(), back_inserter(result));
set_difference(a.begin(), a.end(), b.begin(), b.end(), back_inserter(result));

// Permutations
next_permutation(v.begin(), v.end());
prev_permutation(v.begin(), v.end());

// Shuffle
#include <random>
mt19937 rng(random_device{}());
shuffle(v.begin(), v.end(), rng);
```

### Iterators

```cpp
// Iterator categories (from least to most powerful)
// Input → Output → Forward → Bidirectional → Random Access → Contiguous

vector<int> v = {1, 2, 3, 4, 5};

// Iterator types
v.begin();    // iterator to first
v.end();      // iterator past last
v.rbegin();   // reverse iterator to last
v.rend();     // reverse iterator past first
v.cbegin();   // const_iterator

// Advance & distance
auto it = v.begin();
advance(it, 3);          // Move forward 3 (works for all iterator types)
distance(v.begin(), it); // 3

// Insert iterators
back_inserter(v);         // push_back
front_inserter(lst);      // push_front (list only)
inserter(v, v.begin());   // insert at position
```

---

## 14. Exception Handling

```cpp
#include <stdexcept>

// Basic try-catch
try {
    int x = -1;
    if (x < 0) throw invalid_argument("Negative value");
    // ...
} catch (const invalid_argument& e) {
    cerr << "Invalid argument: " << e.what() << endl;
} catch (const runtime_error& e) {
    cerr << "Runtime error: " << e.what() << endl;
} catch (const exception& e) {
    cerr << "Exception: " << e.what() << endl;
} catch (...) {
    cerr << "Unknown exception!" << endl;
}

// Standard exception hierarchy
// exception
//   ├── logic_error
//   │     ├── invalid_argument
//   │     ├── domain_error
//   │     ├── length_error
//   │     └── out_of_range
//   └── runtime_error
//         ├── range_error
//         ├── overflow_error
//         └── underflow_error

// Custom exception
class DatabaseError : public runtime_error {
    int errorCode;
public:
    DatabaseError(const string& msg, int code)
        : runtime_error(msg), errorCode(code) {}

    int getCode() const { return errorCode; }
};

// Re-throwing
try {
    riskyOperation();
} catch (const exception& e) {
    log(e.what());
    throw;   // Re-throw same exception
}

// noexcept specifier (C++11)
void safeFunc() noexcept { ... }  // Guaranteed not to throw
// If it does throw → std::terminate() is called

// noexcept in move operations (important for performance)
class MyClass {
public:
    MyClass(MyClass&&) noexcept;     // Allows vector to use move
    MyClass& operator=(MyClass&&) noexcept;
};
```

---

## 15. File I/O

```cpp
#include <fstream>
#include <sstream>

// Write to file
ofstream out("output.txt");
if (!out.is_open()) throw runtime_error("Cannot open file");
out << "Line 1\n";
out << "Value: " << 42 << "\n";
out.close();

// Append to file
ofstream app("output.txt", ios::app);
app << "Appended line\n";

// Read from file
ifstream in("input.txt");
string line;
while (getline(in, line)) {
    cout << line << "\n";
}
in.close();

// Read word by word
string word;
while (in >> word) {
    cout << word << " ";
}

// Read entire file
string content((istreambuf_iterator<char>(in)),
                istreambuf_iterator<char>());

// Binary I/O
ofstream binOut("data.bin", ios::binary);
int num = 42;
binOut.write(reinterpret_cast<char*>(&num), sizeof(num));

ifstream binIn("data.bin", ios::binary);
int readNum;
binIn.read(reinterpret_cast<char*>(&readNum), sizeof(readNum));

// File position
in.seekg(0, ios::beg);      // Seek to beginning
in.seekg(0, ios::end);      // Seek to end
in.tellg();                  // Current position

// String streams
ostringstream oss;
oss << "Value: " << 42 << " Price: " << 3.14;
string result = oss.str();

istringstream iss("10 20 30");
int a, b, c;
iss >> a >> b >> c;

// Check state
in.eof();    // At end of file
in.fail();   // Error occurred
in.good();   // No errors
in.clear();  // Clear error flags

// Open modes
ios::in      // Read
ios::out     // Write (truncates)
ios::app     // Append
ios::binary  // Binary mode
ios::ate     // Seek to end on open
ios::trunc   // Truncate on open
// Combine: ios::in | ios::out
```

---

## 16. Modern C++ (C++11/14/17/20)

### C++11 Key Features

```cpp
// nullptr
int* p = nullptr;

// auto type deduction
auto x = 42;
auto it = v.begin();

// Range-based for
for (const auto& item : container) {}

// Lambda expressions
auto fn = [](int x) { return x * 2; };

// Move semantics
string s1 = "hello";
string s2 = move(s1);   // s1 is now empty

// Smart pointers
auto p = make_shared<MyClass>();

// Initializer lists
vector<int> v = {1, 2, 3, 4, 5};

// Uniform initialization
struct Point { int x, y; };
Point p{3, 4};
vector<int> v{1, 2, 3};

// Override / final
class D : public B { void foo() override; };

// Deleted functions
MyClass(const MyClass&) = delete;

// Defaulted functions
MyClass() = default;

// constexpr
constexpr int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n-1);
}

// static_assert
static_assert(sizeof(int) == 4, "int must be 4 bytes");

// noexcept
void safe() noexcept {}

// Scoped enum (enum class)
enum class Color { Red, Green, Blue };
Color c = Color::Red;

// Thread support
#include <thread>
thread t([]{ cout << "hello"; });
t.join();
```

### C++14 Additions

```cpp
// Generic lambdas
auto f = [](auto x, auto y) { return x + y; };

// make_unique
auto p = make_unique<MyClass>(args);

// std::exchange
int old = exchange(x, new_val);   // Sets x = new_val, returns old x

// Return type deduction
auto add(int a, int b) { return a + b; }

// Variable templates
template <typename T>
constexpr T pi = T(3.14159265358979);

double x = pi<double>;
float y = pi<float>;
```

### C++17 Additions

```cpp
// Structured bindings
auto [a, b, c] = make_tuple(1, 2.0, "three");
auto [key, val] = *map.begin();

// if with initializer
if (auto [it, ok] = map.insert({key, val}); ok) {
    cout << "Inserted";
}

// Fold expressions (variadic templates)
template <typename... Args>
auto sum(Args... args) { return (... + args); }

// std::optional
#include <optional>
optional<int> findValue(vector<int>& v, int target) {
    for (int x : v) if (x == target) return x;
    return nullopt;
}
if (auto val = findValue(v, 3)) {
    cout << *val;
}

// std::variant
#include <variant>
variant<int, double, string> v = 42;
v = 3.14;
v = "hello";
get<string>(v);     // "hello"
holds_alternative<string>(v);   // true

// std::any
#include <any>
any a = 42;
a = 3.14;
a = string("hello");
any_cast<string>(a);

// Parallel algorithms
#include <execution>
sort(execution::par, v.begin(), v.end());   // Parallel sort!

// std::filesystem
#include <filesystem>
namespace fs = filesystem;
fs::path p = "/home/user/file.txt";
p.filename();       // "file.txt"
p.extension();      // ".txt"
p.parent_path();    // "/home/user"
fs::exists(p);
fs::create_directory("newdir");
for (auto& entry : fs::directory_iterator(".")) {
    cout << entry.path() << "\n";
}

// string_view
#include <string_view>
string_view sv = "Hello, World";
sv.substr(0, 5);   // No allocation!

// std::byte
#include <cstddef>
byte b{0xFF};
```

### C++20 Additions

```cpp
// Concepts (covered in templates section)

// Ranges
#include <ranges>
auto evens = v | views::filter([](int x){ return x % 2 == 0; })
               | views::transform([](int x){ return x * 2; });
for (int x : evens) cout << x;

vector<int> result;
ranges::copy(evens, back_inserter(result));
ranges::sort(v);

// Coroutines (simplified)
#include <coroutine>
// Complex topic — used for async generators, cooperative multitasking

// std::format
#include <format>
string s = format("Hello, {}! You are {} years old.", name, age);
string hex = format("{:#010x}", 255);   // "0x000000ff"

// Three-way comparison (spaceship)
auto cmp = a <=> b;   // strong_ordering, weak_ordering, or partial_ordering

// Designated initializers
struct Config { int width = 800; int height = 600; bool fullscreen = false; };
Config c{.width = 1920, .height = 1080};

// span (non-owning view over contiguous data)
#include <span>
void process(span<int> data) { ... }
process(v);          // Works with vector
process(arr);        // Works with C array

// jthread (joinable thread with stop token)
#include <thread>
jthread t([](stop_token st) {
    while (!st.stop_requested()) { ... }
});
t.request_stop();    // Request stop
// Automatically joined on destruction
```

---

## 17. Concurrency & Multithreading

```cpp
#include <thread>
#include <mutex>
#include <condition_variable>
#include <atomic>
#include <future>

// Basic thread
thread t1(myFunction);
thread t2([](int x){ cout << x; }, 42);
t1.join();         // Wait for thread to finish
t2.detach();       // Let it run independently (be careful!)
t1.joinable();     // Check before join/detach

// Mutex (mutual exclusion)
mutex mtx;

void safeIncrement(int& count) {
    lock_guard<mutex> lock(mtx);  // Auto-unlock on scope exit
    count++;
}

// unique_lock (more flexible)
void safeOp() {
    unique_lock<mutex> lock(mtx);
    // ... critical section ...
    lock.unlock();    // Can manually unlock
    // ... non-critical ...
    lock.lock();      // Re-lock
}

// Shared mutex (multiple readers, one writer)
#include <shared_mutex>
shared_mutex smtx;

void readData() {
    shared_lock<shared_mutex> lock(smtx);  // Multiple simultaneous readers
    // read...
}

void writeData() {
    unique_lock<shared_mutex> lock(smtx);  // Exclusive writer
    // write...
}

// Condition variable
mutex cv_mtx;
condition_variable cv;
bool ready = false;

// Producer
void producer() {
    lock_guard<mutex> lock(cv_mtx);
    ready = true;
    cv.notify_one();    // or notify_all()
}

// Consumer
void consumer() {
    unique_lock<mutex> lock(cv_mtx);
    cv.wait(lock, []{ return ready; });  // Wait until ready == true
    // Process...
}

// Atomic (lock-free for simple types)
atomic<int> counter{0};
counter++;                        // Thread-safe
counter.fetch_add(1);
counter.load();
counter.store(42);
counter.exchange(100);            // Set and return old value
int expected = 5;
counter.compare_exchange_strong(expected, 10);  // CAS

// Future & Promise (one-time channel between threads)
promise<int> p;
future<int> f = p.get_future();

thread t([&p]{ p.set_value(42); });
int result = f.get();    // Blocks until value is ready

// async (high-level task launching)
#include <future>
future<int> fut = async(launch::async, []{ return 42; });
int val = fut.get();

future<void> f2 = async(launch::async, heavyComputation, param);
// launch::deferred — lazy evaluation, runs when .get() is called

// packaged_task
packaged_task<int(int, int)> task(add);
future<int> fut = task.get_future();
thread t(move(task), 3, 4);
int result = fut.get();   // 7
```

---

## 18. Move Semantics & Perfect Forwarding

### Lvalues and Rvalues

```cpp
// lvalue — has a name, persists beyond expression
int x = 42;      // x is lvalue

// rvalue — temporary, no name
42               // rvalue
x + 1            // rvalue
string("hello")  // rvalue (temporary)

// lvalue reference
int& lref = x;       // OK
// int& r = 42;      // Error!

// rvalue reference (C++11)
int&& rref = 42;     // OK
int&& rref2 = x + 1; // OK
```

### Move Constructor & Move Assignment

```cpp
class Buffer {
    int* data;
    size_t size;

public:
    // Constructor
    Buffer(size_t n) : size(n), data(new int[n]) {}

    // Destructor
    ~Buffer() { delete[] data; }

    // Copy constructor (deep copy)
    Buffer(const Buffer& other) : size(other.size), data(new int[other.size]) {
        copy(other.data, other.data + size, data);
    }

    // Move constructor (steal resources — O(1))
    Buffer(Buffer&& other) noexcept : size(other.size), data(other.data) {
        other.data = nullptr;   // Leave source in valid but empty state
        other.size = 0;
    }

    // Move assignment
    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            delete[] data;          // Free existing resources
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }
};

Buffer b1(100);
Buffer b2 = move(b1);   // Calls move constructor — b1 is now empty
```

### std::move and std::forward

```cpp
// std::move — cast to rvalue reference (doesn't actually move)
string s1 = "hello";
string s2 = move(s1);  // s1 is left in valid but unspecified state

// Perfect forwarding — preserve lvalue/rvalue category
template <typename T>
void wrapper(T&& arg) {
    process(forward<T>(arg));  // forward as lvalue if T is lvalue ref, else rvalue
}

// Universal reference (forwarding reference)
template <typename T>
void f(T&& x) { ... }   // T&& is universal here, not just rvalue ref
```

### Rule of Five

```cpp
// If you define any of these, define all five:
class MyClass {
    ~MyClass();                              // Destructor
    MyClass(const MyClass&);                 // Copy constructor
    MyClass& operator=(const MyClass&);      // Copy assignment
    MyClass(MyClass&&) noexcept;             // Move constructor
    MyClass& operator=(MyClass&&) noexcept;  // Move assignment
};

// Rule of Zero: if possible, let compiler generate all of them
// (use smart pointers and standard containers)
```

---

## 19. Design Patterns in C++

### Singleton

```cpp
class Singleton {
    static Singleton* instance;
    Singleton() {}

public:
    static Singleton& getInstance() {
        static Singleton instance;   // Thread-safe since C++11
        return instance;
    }

    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
};
```

### Factory

```cpp
class Shape { public: virtual void draw() = 0; virtual ~Shape() {} };
class Circle : public Shape { public: void draw() override { cout << "Circle"; } };
class Square : public Shape { public: void draw() override { cout << "Square"; } };

class ShapeFactory {
public:
    static unique_ptr<Shape> create(const string& type) {
        if (type == "circle") return make_unique<Circle>();
        if (type == "square") return make_unique<Square>();
        throw invalid_argument("Unknown shape: " + type);
    }
};

auto s = ShapeFactory::create("circle");
s->draw();
```

### Observer

```cpp
class Observer {
public:
    virtual void update(int value) = 0;
    virtual ~Observer() {}
};

class Subject {
    vector<Observer*> observers;
    int value;

public:
    void attach(Observer* o) { observers.push_back(o); }
    void detach(Observer* o) {
        observers.erase(remove(observers.begin(), observers.end(), o), observers.end());
    }
    void setValue(int v) {
        value = v;
        notify();
    }
    void notify() {
        for (auto* o : observers) o->update(value);
    }
};
```

### CRTP (Curiously Recurring Template Pattern)

```cpp
// Static polymorphism — no vtable overhead
template <typename Derived>
class Base {
public:
    void interface() {
        static_cast<Derived*>(this)->implementation();
    }
};

class Derived : public Base<Derived> {
public:
    void implementation() {
        cout << "Derived implementation";
    }
};
```

### Strategy

```cpp
class SortStrategy {
public:
    virtual void sort(vector<int>& v) = 0;
    virtual ~SortStrategy() {}
};

class BubbleSort : public SortStrategy {
public:
    void sort(vector<int>& v) override { /* bubble sort */ }
};

class QuickSort : public SortStrategy {
public:
    void sort(vector<int>& v) override { /* quick sort */ }
};

class Sorter {
    unique_ptr<SortStrategy> strategy;
public:
    Sorter(unique_ptr<SortStrategy> s) : strategy(move(s)) {}
    void setStrategy(unique_ptr<SortStrategy> s) { strategy = move(s); }
    void sort(vector<int>& v) { strategy->sort(v); }
};
```

---

## 20. Best Practices & Performance

### Code Quality

```cpp
// Prefer const wherever possible
const int MAX = 100;
void process(const vector<int>& v) { }
int getValue() const { return val; }

// Prefer constexpr for compile-time computation
constexpr int fibonacci(int n) {
    return n <= 1 ? n : fibonacci(n-1) + fibonacci(n-2);
}

// Use RAII — don't manage resources manually
// Bad:
FILE* f = fopen("file.txt", "r");
// ... (can leak if exception thrown)
fclose(f);

// Good:
ifstream f("file.txt");
// Automatically closed when f goes out of scope

// Prefer algorithms over raw loops
// Bad:
for (int i = 0; i < v.size(); i++) {
    total += v[i];
}
// Good:
total = accumulate(v.begin(), v.end(), 0);

// Use range-based for
for (const auto& x : v) { }

// Prefer emplace over push/insert
v.emplace_back(args...);    // Construct in-place, no copy
map.emplace(key, value);

// Never return raw owning pointer — return smart pointer or value
unique_ptr<MyClass> createObj() { return make_unique<MyClass>(); }
```

### Performance Tips

```cpp
// Reserve vector capacity when size is known
vector<int> v;
v.reserve(1000);

// Use move semantics
string bigString = buildString();
storeString(move(bigString));

// Avoid unnecessary copies in loops
for (const auto& item : container) { }   // reference, no copy

// Profile before optimizing
// Use: gprof, perf, Valgrind, or compiler's -pg flag

// Avoid virtual functions in hot paths (vtable overhead)
// Consider CRTP for zero-cost abstractions

// Use local variables to avoid repeated pointer dereferences
int* p = &arr[i];
int local = *p;
local += 1;
local += 2;
*p = local;

// Cache-friendly data structures
// Prefer contiguous memory (vector) over linked structures (list)
// Access arrays row-by-row (row-major order in C++)

// Compiler optimizations
// -O2 or -O3 — let compiler do its job
// Use link-time optimization (LTO): -flto
// Use profile-guided optimization (PGO)

// Constexpr to shift computation to compile time
constexpr auto lookup = []() {
    array<int, 256> t{};
    for (int i = 0; i < 256; i++) t[i] = i * i;
    return t;
}();
```

### Common Pitfalls

```cpp
// ❌ Integer overflow (undefined behavior)
int x = INT_MAX;
x++;    // Undefined behavior!
// ✅ Use long long or check before overflow

// ❌ Buffer overflow
char buf[10];
strcpy(buf, "This is way too long!");  // Writes past end
// ✅ Use std::string or strncpy

// ❌ Dangling reference
int& getRef() {
    int local = 42;
    return local;  // local destroyed after function returns!
}
// ✅ Return by value or pointer to heap object

// ❌ Use-after-free
int* p = new int(42);
delete p;
cout << *p;  // Undefined behavior
// ✅ Use smart pointers

// ❌ Double delete
delete p;
delete p;  // Crash
// ✅ Set p = nullptr after delete, or use smart pointers

// ❌ Uninitialized variable
int x;
if (x > 0) { ... }  // Undefined behavior
// ✅ Always initialize: int x = 0;

// ❌ Comparison of signed/unsigned
int size = v.size();  // v.size() returns size_t (unsigned)
// ✅ Use size_t or cast: for (size_t i = 0; i < v.size(); ++i)

// ❌ Slicing
Derived d;
Base b = d;  // Derived parts are sliced off!
// ✅ Use pointers/references to Base: Base& b = d;

// ❌ Forgetting virtual destructor
class Base { ~Base() {} };     // Non-virtual destructor
class Derived : public Base { int* data; ~Derived() { delete data; } };
Base* b = new Derived();
delete b;  // Derived destructor NOT called — memory leak!
// ✅ class Base { virtual ~Base() {} };
```

### Useful Utilities

```cpp
#include <chrono>
// Timing
auto start = chrono::high_resolution_clock::now();
// ... code ...
auto end = chrono::high_resolution_clock::now();
auto duration = chrono::duration_cast<chrono::milliseconds>(end - start);
cout << duration.count() << "ms\n";

#include <random>
// Random numbers
mt19937 rng(random_device{}());
uniform_int_distribution<int> dist(1, 100);
int randNum = dist(rng);

uniform_real_distribution<double> realDist(0.0, 1.0);
double randReal = realDist(rng);

#include <numeric>
// GCD / LCM (C++17)
gcd(12, 8);    // 4
lcm(4, 6);     // 12

// iota
vector<int> v(10);
iota(v.begin(), v.end(), 0);   // {0,1,2,3,4,5,6,7,8,9}

#include <limits>
numeric_limits<int>::max();      // INT_MAX
numeric_limits<double>::min();   // Smallest positive double
numeric_limits<int>::is_signed;  // true
```

---

## Quick Reference Card

### Most-Used STL Headers

| Header | Contents |
|--------|----------|
| `<iostream>` | `cin`, `cout`, `cerr` |
| `<string>` | `std::string` |
| `<vector>` | `std::vector` |
| `<map>` | `std::map`, `multimap` |
| `<unordered_map>` | `std::unordered_map` |
| `<set>` | `std::set`, `multiset` |
| `<algorithm>` | `sort`, `find`, `transform`, etc. |
| `<numeric>` | `accumulate`, `iota`, `gcd` |
| `<functional>` | `function`, `bind`, functors |
| `<memory>` | `unique_ptr`, `shared_ptr` |
| `<thread>` | `thread`, `mutex` |
| `<future>` | `future`, `promise`, `async` |
| `<chrono>` | Time utilities |
| `<filesystem>` | File system operations |
| `<optional>` | `std::optional` |
| `<variant>` | `std::variant` |
| `<any>` | `std::any` |
| `<format>` | `std::format` (C++20) |
| `<ranges>` | Ranges and views (C++20) |
| `<concepts>` | Concepts (C++20) |
| `<cmath>` | `sqrt`, `pow`, `sin`, `cos`, etc. |
| `<cstdlib>` | `abs`, `rand`, `exit` |
| `<cassert>` | `assert` macro |
| `<climits>` | `INT_MAX`, `INT_MIN`, etc. |

### Compile-Time Cheat Sheet

```bash
# Basic compile
g++ -std=c++20 -Wall -Wextra -O2 main.cpp -o app

# Debug build
g++ -std=c++20 -g -fsanitize=address,undefined main.cpp -o app_debug

# Check for memory issues (Valgrind)
valgrind --leak-check=full ./app

# Generate assembly
g++ -std=c++17 -O2 -S main.cpp -o main.s

# Show preprocessed output
g++ -E main.cpp -o main.i
```
