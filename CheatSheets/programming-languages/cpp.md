# C++: A Complete Progressive Tutorial

---

## 1. What & Why

C++ is a multi-paradigm compiled language built on top of C that adds object-oriented, generic, and functional programming features. It maintains C's control over memory and performance while providing powerful abstractions: classes, templates, the standard library, and modern features like smart pointers, lambdas, and ranges.

Why C++ in a world with safer languages? Because no other language combines C's raw performance with abstraction capabilities at scale. The operating system, game engines (Unreal, Unity), browsers (Chrome, Firefox), databases (MySQL, RocksDB), and high-frequency trading systems are C++ because they need both fine-grained control and large codebases. Rust is an alternative for new systems code, but the existing C++ ecosystem is enormous.

Modern C++ (C++17/20) is a very different language from C++98. The language has evolved substantially to fix its most dangerous features while retaining backward compatibility.

---

## 2. Mental Model

C++ extends C with three major additions: classes (data + behavior bundled), templates (generic programming), and the standard library (containers, algorithms, smart pointers).

```
C foundation:      raw memory, pointers, manual allocation
      +
Classes:           encapsulate data + behavior, RAII, inheritance, polymorphism
      +
Templates:         generic code that works with any type, zero-cost abstraction
      +
Standard Library:  vector, map, string, algorithm, smart pointers, threads

RAII (Resource Acquisition Is Initialization):
  Constructor acquires resources (open file, allocate memory)
  Destructor releases resources (close file, free memory)
  When an object goes out of scope, its destructor runs — automatically.
  This eliminates most memory leaks and resource leaks.

Ownership model:
  Stack objects: destroyed automatically when scope ends
  Unique ownership: std::unique_ptr — one owner, destroyed with the pointer
  Shared ownership: std::shared_ptr — reference-counted
  Non-owning reference: raw pointer or reference (be careful)
```

---

## 3. Progressive Examples

### Level 1: Classes, RAII, and the Big Five

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <memory>    // smart pointers
#include <stdexcept> // exceptions

// A well-formed class in modern C++
class DynamicArray {
private:
    int *data_;
    size_t size_;
    size_t capacity_;

public:
    // Constructor: acquires resource
    explicit DynamicArray(size_t initial_capacity = 8)
        : data_(new int[initial_capacity])   // member initializer list
        , size_(0)
        , capacity_(initial_capacity) {}

    // Destructor: releases resource (called automatically when object goes out of scope)
    ~DynamicArray() {
        delete[] data_;   // [] because data_ is an array
    }

    // Copy constructor
    DynamicArray(const DynamicArray& other)
        : data_(new int[other.capacity_])
        , size_(other.size_)
        , capacity_(other.capacity_) {
        std::copy(other.data_, other.data_ + size_, data_);
    }

    // Move constructor (transfers ownership, leaves other in valid but empty state)
    DynamicArray(DynamicArray&& other) noexcept
        : data_(other.data_), size_(other.size_), capacity_(other.capacity_) {
        other.data_ = nullptr;   // prevent double-free
        other.size_ = other.capacity_ = 0;
    }

    // Copy assignment operator
    DynamicArray& operator=(const DynamicArray& other) {
        if (this != &other) {
            delete[] data_;
            data_ = new int[other.capacity_];
            size_ = other.size_;
            capacity_ = other.capacity_;
            std::copy(other.data_, other.data_ + size_, data_);
        }
        return *this;
    }

    // Move assignment operator
    DynamicArray& operator=(DynamicArray&& other) noexcept {
        if (this != &other) {
            delete[] data_;
            data_ = other.data_;
            size_ = other.size_;
            capacity_ = other.capacity_;
            other.data_ = nullptr;
        }
        return *this;
    }

    void push_back(int val) {
        if (size_ == capacity_) {
            capacity_ *= 2;
            int *new_data = new int[capacity_];
            std::copy(data_, data_ + size_, new_data);
            delete[] data_;
            data_ = new_data;
        }
        data_[size_++] = val;
    }

    int& operator[](size_t idx) {
        if (idx >= size_) throw std::out_of_range("Index out of range");
        return data_[idx];
    }

    size_t size() const { return size_; }

    // Iterator support (for range-based for loops)
    int* begin() { return data_; }
    int* end()   { return data_ + size_; }
};

// Usage — no manual memory management needed
int main() {
    DynamicArray arr;
    arr.push_back(10);
    arr.push_back(20);
    arr.push_back(30);

    for (int val : arr) {  // range-based for loop — uses begin()/end()
        std::cout << val << " ";
    }
    // arr is destroyed here — ~DynamicArray() called automatically
}
```

### Level 2: Templates and Generic Programming

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
#include <type_traits>
#include <concepts>   // C++20

// Function template: works with any type that supports operator<
template <typename T>
T max_of(T a, T b) {
    return (a > b) ? a : b;
}

// Usage:
auto m1 = max_of(3, 5);           // T deduced as int
auto m2 = max_of(3.14, 2.72);     // T deduced as double
auto m3 = max_of<int>(3.9, 2.1);  // T explicitly int (truncates to 3, 2)

// Class template: Stack<T> works with any type
template <typename T>
class Stack {
    std::vector<T> data_;   // delegates to std::vector — RAII handled automatically
public:
    void push(T val) { data_.push_back(std::move(val)); }
    T pop() {
        if (data_.empty()) throw std::underflow_error("Stack empty");
        T top = std::move(data_.back());
        data_.pop_back();
        return top;
    }
    T& top()            { return data_.back(); }
    bool empty() const  { return data_.empty(); }
    size_t size() const { return data_.size(); }
};

// Template specialization: different behavior for char*
template<>
class Stack<bool> {  // specialized version for booleans
    std::vector<uint8_t> data_;  // pack multiple bools (space optimization)
public:
    void push(bool val) { data_.push_back(val ? 1 : 0); }
    bool pop() { bool v = data_.back(); data_.pop_back(); return v; }
};

// C++20 Concepts: constrain template types explicitly
template <typename T>
concept Numeric = std::is_arithmetic_v<T>;

template <Numeric T>
T safe_divide(T a, T b) {
    if (b == T{}) throw std::domain_error("Division by zero");
    return a / b;
}

// Variadic templates
template <typename... Args>
void print_all(Args&&... args) {
    ((std::cout << std::forward<Args>(args) << " "), ...);  // fold expression
    std::cout << '\n';
}

print_all(1, "hello", 3.14, true);   // 1 hello 3.14 1
```

### Level 3: Modern C++ Idioms

```cpp
#include <vector>
#include <algorithm>
#include <numeric>
#include <ranges>     // C++20
#include <functional>
#include <optional>
#include <variant>
#include <string_view>

// Smart pointers — prefer over raw pointers
#include <memory>

// Unique ownership: one owner, automatically freed
std::unique_ptr<int> up = std::make_unique<int>(42);
*up = 100;
// up->something() for objects
// up is destroyed at end of scope, frees the int

// Shared ownership: reference-counted
auto sp1 = std::make_shared<std::vector<int>>(3, 0);  // shared_ptr
auto sp2 = sp1;   // both sp1 and sp2 point to same vector; ref count = 2
sp1.reset();      // ref count drops to 1; sp2 still valid
// sp2 destroyed: ref count = 0, vector freed

// Lambdas: anonymous functions, can capture from enclosing scope
auto add = [](int a, int b) { return a + b; };
std::cout << add(3, 4) << '\n';  // 7

int multiplier = 10;
auto multiply = [multiplier](int x) { return x * multiplier; };  // capture by value
auto scale     = [&multiplier](int x) { return x * multiplier; };  // capture by reference

// Standard algorithms with lambdas
std::vector<int> nums = {5, 2, 8, 1, 9, 3, 7, 4, 6};

std::sort(nums.begin(), nums.end());                    // ascending
std::sort(nums.begin(), nums.end(), std::greater<int>{}); // descending
std::sort(nums.begin(), nums.end(), [](int a, int b) { return a > b; }); // same

auto it = std::find(nums.begin(), nums.end(), 7);
if (it != nums.end()) std::cout << "Found 7 at index " << std::distance(nums.begin(), it) << '\n';

auto count_even = std::count_if(nums.begin(), nums.end(), [](int n) { return n % 2 == 0; });

// C++20 Ranges: composable, lazy pipelines
auto evens_doubled = nums
    | std::views::filter([](int n) { return n % 2 == 0; })
    | std::views::transform([](int n) { return n * 2; });

for (int v : evens_doubled) std::cout << v << ' ';

// Optional: represent "value or nothing" without null pointers
std::optional<int> find_value(const std::vector<int>& v, int target) {
    auto it = std::find(v.begin(), v.end(), target);
    if (it == v.end()) return std::nullopt;
    return *it;
}

auto result = find_value(nums, 7);
if (result) std::cout << "Found: " << *result << '\n';
// OR: result.value_or(-1)  → -1 if not found

// Variant: type-safe union
std::variant<int, std::string, double> var;
var = 42;          // holds int
var = "hello";     // now holds string
var = 3.14;        // now holds double

std::visit([](auto&& val) {
    std::cout << val << '\n';
}, var);
```

### Level 4: Inheritance and Polymorphism

```cpp
#include <iostream>
#include <memory>
#include <vector>
#include <string>

// Base class with virtual functions
class Shape {
protected:
    std::string color_;

public:
    explicit Shape(std::string color) : color_(std::move(color)) {}
    virtual ~Shape() = default;   // ALWAYS virtual in polymorphic base classes!

    virtual double area() const = 0;         // pure virtual: must override
    virtual std::string describe() const {   // virtual: can override
        return "Shape: " + color_;
    }

    std::string color() const { return color_; }
};

class Circle : public Shape {
    double radius_;
public:
    Circle(std::string color, double radius)
        : Shape(std::move(color)), radius_(radius) {}

    double area() const override {           // override: enforces that we're overriding
        return 3.14159 * radius_ * radius_;
    }

    std::string describe() const override {
        return "Circle (r=" + std::to_string(radius_) + "): " + color();
    }
};

class Rectangle : public Shape {
    double width_, height_;
public:
    Rectangle(std::string color, double w, double h)
        : Shape(std::move(color)), width_(w), height_(h) {}

    double area() const override { return width_ * height_; }
};

// Dynamic dispatch through base class pointer
void print_shape_info(const Shape& s) {
    std::cout << s.describe() << " area=" << s.area() << '\n';
}

int main() {
    // Polymorphism: shapes stored as base-class smart pointers
    std::vector<std::unique_ptr<Shape>> shapes;
    shapes.push_back(std::make_unique<Circle>("red", 5.0));
    shapes.push_back(std::make_unique<Rectangle>("blue", 4.0, 6.0));

    double total_area = 0;
    for (const auto& shape : shapes) {
        print_shape_info(*shape);
        total_area += shape->area();
    }
    std::cout << "Total area: " << total_area << '\n';

    // Dynamic cast: safely downcast when needed
    Shape *s = shapes[0].get();
    if (auto *circle = dynamic_cast<Circle*>(s)) {
        std::cout << "It's a circle!\n";
    }
}
```

### Level 5: Exception Safety and Error Handling

```cpp
#include <stdexcept>
#include <fstream>
#include <string>

// RAII for exception safety: resources released even when exceptions thrown
class FileGuard {
    FILE *file_;
public:
    explicit FileGuard(const char *path, const char *mode) {
        file_ = fopen(path, mode);
        if (!file_) throw std::runtime_error(std::string("Cannot open: ") + path);
    }
    ~FileGuard() { if (file_) fclose(file_); }   // auto-closes on scope exit

    FILE* get() { return file_; }
    // Deleted to prevent copying (ownership transfer only)
    FileGuard(const FileGuard&) = delete;
    FileGuard& operator=(const FileGuard&) = delete;
};

// Exception safety levels:
// Basic:    no resource leaks, valid (but possibly changed) state
// Strong:   if exception thrown, state unchanged (transactional)
// No-throw: guaranteed not to throw (noexcept)

// Strong exception guarantee using copy-and-swap idiom
void update_config(std::vector<std::string>& config, const std::string& new_entry) {
    std::vector<std::string> temp = config;   // copy
    temp.push_back(new_entry);                // modify copy
    config = std::move(temp);                 // move into config (noexcept swap)
}  // if push_back throws, config is unchanged — strong guarantee

// Custom exception hierarchy
struct AppError : std::runtime_error {
    explicit AppError(const std::string& msg) : std::runtime_error(msg) {}
};

struct DatabaseError : AppError {
    explicit DatabaseError(const std::string& msg) : AppError("DB: " + msg) {}
};

// Error handling in practice
std::string read_config(const std::string& path) {
    try {
        FileGuard f(path.c_str(), "r");
        char buf[4096];
        std::string content;
        while (fgets(buf, sizeof(buf), f.get())) {
            content += buf;
        }
        return content;
    }
    catch (const std::runtime_error& e) {
        throw AppError("Config error: " + std::string(e.what()));
    }
}
```

### Level 6: Move Semantics and Performance

```cpp
#include <vector>
#include <string>
#include <utility>   // std::move, std::forward
#include <chrono>

// Move semantics: transfer ownership instead of copying
// Key insight: temporary objects (rvalues) can be "stolen" from

std::vector<int> expensive_compute() {
    std::vector<int> result(1000000);
    // ... fill result ...
    return result;   // NRVO or move semantics — no copy!
}

// Without move semantics (C++03): returns a copy (1M int copies!)
// With move semantics (C++11+): moves the internal buffer — O(1)!

auto data = expensive_compute();  // moved, not copied

// std::move: cast to rvalue reference (enables move)
std::string s1 = "hello world";
std::string s2 = std::move(s1);  // s2 gets the buffer, s1 is now empty
// s1 is in a "valid but unspecified state" after move — don't use it!

// Perfect forwarding: preserve value category when passing through templates
template <typename T>
void wrapper(T&& arg) {
    some_function(std::forward<T>(arg));  // lvalue → lvalue, rvalue → rvalue
}

// When to use std::move:
// 1. Returning local variables (usually automatic — NRVO)
// 2. Storing in containers: data.push_back(std::move(large_string));
// 3. Passing to functions that will store the argument
// 4. Inside move constructors and move assignment operators

// Performance measurement
void benchmark() {
    using namespace std::chrono;
    constexpr int N = 10000;

    // Copy-based (slow)
    auto start = high_resolution_clock::now();
    std::vector<std::string> with_copy;
    for (int i = 0; i < N; i++) {
        std::string s = "string_value_" + std::to_string(i);
        with_copy.push_back(s);       // copies s into the vector
    }
    auto end = high_resolution_clock::now();
    std::cout << "Copy: " << duration_cast<microseconds>(end-start).count() << "us\n";

    // Move-based (fast)
    start = high_resolution_clock::now();
    std::vector<std::string> with_move;
    for (int i = 0; i < N; i++) {
        std::string s = "string_value_" + std::to_string(i);
        with_move.push_back(std::move(s));  // moves s into the vector
    }
    end = high_resolution_clock::now();
    std::cout << "Move: " << duration_cast<microseconds>(end-start).count() << "us\n";
    // Move is typically 2-10x faster for string-heavy operations
}
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Forgetting `virtual` on base class destructor**

```cpp
// WRONG: derived class destructor won't be called when deleting through base pointer!
class Base { public: ~Base() {} };
class Derived : public Base { public: ~Derived() { cleanup(); } };

Base *b = new Derived();
delete b;   // ONLY ~Base() called! Derived's resources leaked!

// CORRECT: virtual destructor in any class that will be inherited
class Base { public: virtual ~Base() = default; };
// Now delete b correctly calls ~Derived() then ~Base()
```

**Mistake 2: Using raw pointers for ownership**

```cpp
// WRONG: manual memory management — prone to leaks and double-frees
MyObject *obj = new MyObject();
process(obj);   // if process throws, obj leaks!
delete obj;

// CORRECT: smart pointers — RAII handles cleanup automatically
auto obj = std::make_unique<MyObject>();
process(*obj);  // if throws, unique_ptr destructor frees memory
```

**Mistake 3: Moving from and then using an object**

```cpp
std::string s = "hello";
std::string t = std::move(s);   // s is now in valid but unspecified state
std::cout << s.size();          // UB: s has been moved from, should not be used!

// CORRECT: don't use moved-from variables
std::string t = std::move(s);
// Either reassign before using:
s = "new value";
// Or don't use s again
```

**Mistake 4: Not using `override` keyword**

```cpp
struct Base { virtual void foo() const; };

// WRONG: typo creates a NEW function, not an override!
struct Derived : Base { void foo();  };  // missing const — different signature!

// CORRECT: override catches this at compile time
struct Derived : Base { void foo() const override; };  // error if not matching base
```

---

## 5. The "Why Does This Work" Layer

### How Virtual Dispatch Works (vtable)

Every class with virtual functions has a hidden vtable — a table of function pointers, one per virtual function. Every object of that class has a hidden pointer (vptr) to its class's vtable. When you call `shape->area()`, the compiler generates: dereference vptr → look up `area` slot in vtable → call through that pointer. This is why virtual calls have a small overhead (one indirection) and why `final` classes allow devirtualization (compiler can see no overrides exist).

### Why Move Semantics Exist

Before C++11, returning a `std::vector<int>` from a function meant copying the entire vector's heap-allocated buffer. With move semantics, the compiler can "steal" the buffer from a temporary (rvalue) vector instead. The move constructor takes the pointer to the heap buffer and sets the source's pointer to null. No data is copied — just a pointer is transferred. For large vectors, this is the difference between microseconds and nanoseconds.

---

## 6. Quick Reference

### Modern C++ Compilation

```bash
g++ -std=c++17 -Wall -Wextra -O2 -o app main.cpp
g++ -std=c++20 -Wall -Wextra -O0 -g -fsanitize=address,undefined -o app_debug main.cpp
```

### Smart Pointers

```cpp
// Unique ownership
auto p = std::make_unique<T>(args...);
p->method();   // use
T& ref = *p;   // dereference
// Automatic delete when p goes out of scope

// Shared ownership
auto sp = std::make_shared<T>(args...);
auto sp2 = sp;  // both own the object
// Freed when last shared_ptr is destroyed

// Non-owning reference
std::weak_ptr<T> wp = sp;  // observe shared_ptr without ownership
if (auto locked = wp.lock()) { /* use locked */ }
```

### Key Standard Library

```cpp
// Containers
std::vector<T>       // dynamic array
std::list<T>         // doubly-linked list
std::map<K,V>        // sorted map (BST), O(log n)
std::unordered_map<K,V>  // hash map, O(1) average
std::set<T>          // sorted set
std::stack/queue/priority_queue<T>

// Algorithms (in <algorithm>)
std::sort, std::find, std::find_if, std::count_if
std::transform, std::copy, std::for_each
std::min_element, std::max_element
std::accumulate (in <numeric>)

// Utilities
std::optional<T>     // value or nothing
std::variant<T...>   // type-safe union
std::string_view     // non-owning string reference (no copy)
std::span<T>         // non-owning view over contiguous data
```
