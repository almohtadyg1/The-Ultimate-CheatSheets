# CMake Complete Reference Guide
> From Scratch to Expert Level — Everything You Need to Master CMake

---

## Table of Contents

1. [What is CMake & How It Works](#1-what-is-cmake--how-it-works)
2. [Installation & Setup](#2-installation--setup)
3. [Your First CMakeLists.txt](#3-your-first-cmakeliststxt)
4. [Project Structure & Basics](#4-project-structure--basics)
5. [Targets — Executables & Libraries](#5-targets--executables--libraries)
6. [Target Properties & Visibility](#6-target-properties--visibility)
7. [Variables & Cache](#7-variables--cache)
8. [Control Flow](#8-control-flow)
9. [Functions & Macros](#9-functions--macros)
10. [Finding External Dependencies](#10-finding-external-dependencies)
11. [FetchContent & CPM (Dependency Management)](#11-fetchcontent--cpm-dependency-management)
12. [Installing & Packaging](#12-installing--packaging)
13. [Testing with CTest](#13-testing-with-ctest)
14. [Generator Expressions](#14-generator-expressions)
15. [Toolchains & Cross-Compilation](#15-toolchains--cross-compilation)
16. [CMake Presets](#16-cmake-presets)
17. [Useful Built-in Modules](#17-useful-built-in-modules)
18. [Real-World Project Layout](#18-real-world-project-layout)
19. [Best Practices & Common Pitfalls](#19-best-practices--common-pitfalls)
20. [Quick Reference Card](#20-quick-reference-card)

---

## 1. What is CMake & How It Works

CMake is a **build system generator** — it doesn't build your code directly. Instead it reads your `CMakeLists.txt` files and generates native build files for whatever build system you're using (Make, Ninja, Visual Studio, Xcode, etc.).

### The Three-Step Workflow

```
Source Dir              Build Dir              Binaries
CMakeLists.txt  ──►  Makefile/build.ninja  ──►  myapp
                   (cmake ..)               (cmake --build .)
   Configure           Generate                  Build
```

**Step 1 — Configure:** CMake reads all `CMakeLists.txt` files, checks the system, finds dependencies, and stores everything in `CMakeCache.txt`.

**Step 2 — Generate:** CMake writes the actual build files for your chosen generator (Makefile, Ninja, etc.).

**Step 3 — Build:** Your native build tool (make/ninja/msbuild) compiles and links the code.

### Key Concepts

| Concept | What It Means |
|---------|--------------|
| **Target** | Something CMake builds — an executable, library, or custom command |
| **Generator** | The build system CMake writes files for (Ninja, Make, VS, etc.) |
| **Cache** | Persistent variable store in `CMakeCache.txt` — survives reconfigures |
| **Toolchain** | The compiler, linker, and related tools |
| **Source dir** | Where your `CMakeLists.txt` lives (`CMAKE_SOURCE_DIR`) |
| **Build dir** | Where CMake generates files — always separate from source |
| **Property** | Metadata attached to a target, file, or directory |

---

## 2. Installation & Setup

### Installing CMake

**Linux**
```bash
# Ubuntu/Debian
sudo apt install cmake

# Fedora/RHEL
sudo dnf install cmake

# Latest version via pip
pip install cmake

# Or download from cmake.org and add to PATH
```

**macOS**
```bash
brew install cmake
```

**Windows**
- Download installer from [cmake.org/download](https://cmake.org/download)
- Or via `winget install Kitware.CMake`
- Or via `choco install cmake`

**Check version**
```bash
cmake --version
```

### Choosing a Generator

```bash
# List available generators on your system
cmake --help

# Use Ninja (fastest, recommended)
cmake -G Ninja ..

# Use Unix Makefiles (default on Linux/macOS)
cmake -G "Unix Makefiles" ..

# Use Visual Studio (Windows)
cmake -G "Visual Studio 17 2022" -A x64 ..

# Use Xcode (macOS)
cmake -G Xcode ..
```

### Basic Build Workflow

```bash
# 1. Create and enter build directory (out-of-source build — always do this!)
mkdir build && cd build

# 2. Configure
cmake ..

# 3. Build
cmake --build .

# 4. Install (optional)
cmake --install . --prefix /usr/local

# All in one (CMake 3.20+)
cmake -S . -B build
cmake --build build
cmake --install build
```

### Build Types

```bash
cmake -DCMAKE_BUILD_TYPE=Debug   ..   # Debug info, no optimization
cmake -DCMAKE_BUILD_TYPE=Release ..   # Optimized, no debug info
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo ..   # Optimized + debug info
cmake -DCMAKE_BUILD_TYPE=MinSizeRel  ..   # Optimize for size
```

---

## 3. Your First CMakeLists.txt

```cmake
# ALWAYS the first line — minimum CMake version required
cmake_minimum_required(VERSION 3.16)

# Declare the project (name, version, languages)
project(HelloWorld VERSION 1.0.0 LANGUAGES CXX)

# Set C++ standard — use target properties, not global flags
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)    # Use -std=c++17, not -std=gnu++17

# Create an executable target from source files
add_executable(hello main.cpp)
```

```cpp
// main.cpp
#include <iostream>

int main() {
    std::cout << "Hello from CMake!\n";
    return 0;
}
```

```bash
cmake -S . -B build
cmake --build build
./build/hello
```

> **Why `cmake_minimum_required` first?** It sets the policy version, which controls CMake's behavior for backwards compatibility. Always put it as the very first line.

---

## 4. Project Structure & Basics

### Recommended Directory Layout

```
MyProject/
├── CMakeLists.txt          ← Root (top-level) CMakeLists
├── CMakePresets.json       ← Build presets (CMake 3.19+)
├── cmake/
│   └── FindMyLib.cmake     ← Custom Find modules
├── src/
│   ├── CMakeLists.txt
│   ├── main.cpp
│   └── app/
│       ├── CMakeLists.txt
│       └── app.cpp
├── lib/
│   ├── CMakeLists.txt
│   ├── mylib.cpp
│   └── mylib.h
├── include/
│   └── mylib/
│       └── mylib.h         ← Public headers
├── tests/
│   ├── CMakeLists.txt
│   └── test_mylib.cpp
├── docs/
└── third_party/
    └── ...
```

### Subdirectories

```cmake
# Root CMakeLists.txt
cmake_minimum_required(VERSION 3.20)
project(MyProject VERSION 2.1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Each subdirectory has its own CMakeLists.txt
add_subdirectory(lib)      # Processes lib/CMakeLists.txt first
add_subdirectory(src)      # Then src/CMakeLists.txt

option(BUILD_TESTS "Build test suite" OFF)
if(BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()
```

### The `project()` Command

```cmake
project(
    MyProject
    VERSION      2.1.0          # Sets MY_PROJECT_VERSION, _MAJOR, _MINOR, _PATCH
    DESCRIPTION  "A great app"  # Sets MY_PROJECT_DESCRIPTION
    HOMEPAGE_URL "https://example.com"
    LANGUAGES    CXX C          # Languages to enable (checks for compilers)
)

# After project(), these are available:
# PROJECT_NAME             → "MyProject"
# PROJECT_VERSION          → "2.1.0"
# PROJECT_VERSION_MAJOR    → "2"
# PROJECT_VERSION_MINOR    → "1"
# PROJECT_VERSION_PATCH    → "0"
# CMAKE_PROJECT_NAME       → Top-level project name
```

### Useful Built-in Variables

```cmake
CMAKE_SOURCE_DIR          # Top-level source directory (where root CMakeLists.txt is)
CMAKE_BINARY_DIR          # Top-level build directory
CMAKE_CURRENT_SOURCE_DIR  # Current CMakeLists.txt's source directory
CMAKE_CURRENT_BINARY_DIR  # Current CMakeLists.txt's build directory

CMAKE_CXX_COMPILER        # Path to C++ compiler
CMAKE_C_COMPILER          # Path to C compiler
CMAKE_BUILD_TYPE          # Debug / Release / etc.

PROJECT_SOURCE_DIR        # Most recent project()'s source dir
PROJECT_BINARY_DIR        # Most recent project()'s binary dir

CMAKE_INSTALL_PREFIX      # Installation prefix (default: /usr/local)
BUILD_SHARED_LIBS         # Global flag: build shared libs by default
```

---

## 5. Targets — Executables & Libraries

Targets are the **core concept** of modern CMake. Everything revolves around targets and their properties.

### Executables

```cmake
# Single source file
add_executable(myapp main.cpp)

# Multiple source files
add_executable(myapp
    main.cpp
    src/app.cpp
    src/utils.cpp
)

# Using a variable
set(APP_SOURCES
    main.cpp
    src/app.cpp
    src/utils.cpp
)
add_executable(myapp ${APP_SOURCES})

# Glob (convenient but not recommended — CMake won't reconfigure on new files)
file(GLOB_RECURSE APP_SOURCES "src/*.cpp")
add_executable(myapp ${APP_SOURCES})

# Alias target (useful for consistent naming in libraries)
add_executable(MyProject::app ALIAS myapp)
```

### Libraries

```cmake
# Static library (.a / .lib)
add_library(mylib STATIC lib.cpp)

# Shared library (.so / .dylib / .dll)
add_library(mylib SHARED lib.cpp)

# Object library (compiled objects, no linking)
add_library(myobjs OBJECT obj.cpp)

# Header-only (interface) library — no source files
add_library(myheaders INTERFACE)

# Module library (for dlopen, not for linking)
add_library(myplugin MODULE plugin.cpp)

# Let BUILD_SHARED_LIBS variable decide static vs shared
add_library(mylib lib.cpp)

# Alias — create a namespaced alias (mimics installed targets)
add_library(MyProject::mylib ALIAS mylib)
```

### Linking Targets Together

```cmake
# Link mylib to myapp — the core linking command
target_link_libraries(myapp PRIVATE mylib)

# Link multiple libraries
target_link_libraries(myapp
    PRIVATE
        mylib
        fmt::fmt
    PUBLIC
        spdlog::spdlog
    INTERFACE
        Threads::Threads
)

# Link system libraries
target_link_libraries(myapp PRIVATE pthread m dl)
```

---

## 6. Target Properties & Visibility

This is the **most important concept** to understand in modern CMake. Every `target_*` command takes a visibility keyword.

### Visibility Keywords

| Keyword | Meaning |
|---------|---------|
| `PRIVATE` | Only this target uses this property. Not propagated to dependents. |
| `PUBLIC` | This target AND any target linking to it gets this property. |
| `INTERFACE` | Only propagated to targets that link to this target. Not used by this target itself. |

```
        PRIVATE           PUBLIC             INTERFACE
        ───────           ──────             ─────────
mylib:  uses it       uses it            does NOT use it
myapp   (links to     gets it            gets it
         mylib):      automatically      automatically
        does NOT get
```

### Include Directories

```cmake
# myapp needs headers from include/
target_include_directories(myapp PRIVATE include/)

# mylib's headers are needed by mylib AND anyone linking to it
target_include_directories(mylib PUBLIC include/)

# myheaders (header-only): only propagate, don't use yourself
target_include_directories(myheaders INTERFACE include/)

# Generator expression: use source tree during build, install tree after install
target_include_directories(mylib PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
```

### Compile Definitions

```cmake
# Define a preprocessor macro
target_compile_definitions(myapp PRIVATE APP_VERSION="1.0")

# Conditional definition
target_compile_definitions(mylib PUBLIC
    $<$<CONFIG:Debug>:DEBUG_BUILD>
    $<$<CONFIG:Release>:NDEBUG>
)

# Platform-specific
target_compile_definitions(mylib PRIVATE
    $<$<PLATFORM_ID:Windows>:WIN32_LEAN_AND_MEAN>
    $<$<PLATFORM_ID:Linux>:_GNU_SOURCE>
)
```

### Compile Options

```cmake
# Add compiler flags to a specific target (not globally!)
target_compile_options(myapp PRIVATE
    -Wall
    -Wextra
    -Wpedantic
)

# Cross-compiler flags using generator expressions
target_compile_options(mylib PRIVATE
    $<$<CXX_COMPILER_ID:MSVC>:/W4 /WX>
    $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -Werror>
)

# Optimization flags by build type
target_compile_options(myapp PRIVATE
    $<$<CONFIG:Debug>:-O0 -g3>
    $<$<CONFIG:Release>:-O3 -march=native>
)
```

### C++ Standard per Target

```cmake
# Better than global CMAKE_CXX_STANDARD — set per target
set_target_properties(myapp PROPERTIES
    CXX_STANDARD          20
    CXX_STANDARD_REQUIRED ON
    CXX_EXTENSIONS        OFF
)

# Or use target_compile_features (most portable)
target_compile_features(myapp PRIVATE cxx_std_20)
target_compile_features(mylib PUBLIC cxx_std_17)
```

### Getting/Setting Properties Directly

```cmake
# Set any property
set_target_properties(myapp PROPERTIES
    OUTPUT_NAME       "my-application"    # Binary filename
    VERSION           "1.2.3"
    SOVERSION         "1"                 # Shared lib soname
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
    LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
    ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
)

# Get a property
get_target_property(OUT_DIR myapp RUNTIME_OUTPUT_DIRECTORY)
message(STATUS "Output dir: ${OUT_DIR}")
```

---

## 7. Variables & Cache

### Regular Variables

```cmake
# Set a variable
set(MY_VAR "hello")
set(MY_LIST a b c d)         # List: semicolon-separated internally
set(MY_LIST "a;b;c;d")       # Same as above

# Use a variable
message(STATUS "Value: ${MY_VAR}")

# Unset a variable
unset(MY_VAR)

# Variable scope: variables are scoped to the current CMakeLists.txt
# and its children via add_subdirectory()
set(VAR "parent")
add_subdirectory(child)   # child sees VAR = "parent"
# child cannot change parent's VAR unless using PARENT_SCOPE

# In child/CMakeLists.txt:
set(VAR "child_value" PARENT_SCOPE)   # Propagate up to parent
```

### Lists

```cmake
set(MY_LIST alpha beta gamma)

# Append
list(APPEND MY_LIST delta epsilon)

# Get length
list(LENGTH MY_LIST len)

# Get element
list(GET MY_LIST 0 first)    # first = "alpha"

# Remove element
list(REMOVE_ITEM MY_LIST beta)

# Find element (index)
list(FIND MY_LIST gamma idx) # idx = index, or -1 if not found

# Sort
list(SORT MY_LIST)

# Reverse
list(REVERSE MY_LIST)

# Join to string
list(JOIN MY_LIST ", " result)  # result = "alpha, gamma, delta, epsilon"

# Iterate
foreach(item IN LISTS MY_LIST)
    message(STATUS "Item: ${item}")
endforeach()
```

### Cache Variables

Cache variables are stored in `CMakeCache.txt` and persist between `cmake` runs. They're how you expose options to users.

```cmake
# Syntax: set(VAR value CACHE TYPE "docstring" [FORCE])

set(MY_OPTION "default" CACHE STRING "My configurable option")
set(PORT_NUMBER 8080 CACHE STRING "Server port")
set(BUILD_TESTS OFF CACHE BOOL "Build the test suite")
set(LIB_PATH "/usr/lib" CACHE PATH "Path to libraries")
set(HEADER_FILE "/usr/include/foo.h" CACHE FILEPATH "Path to header")

# FORCE — override even if already in cache
set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Build type" FORCE)

# option() is shorthand for CACHE BOOL
option(ENABLE_LOGGING "Enable logging support" ON)
# Equivalent to:
set(ENABLE_LOGGING ON CACHE BOOL "Enable logging support")

# Mark as advanced (hidden in cmake-gui by default)
mark_as_advanced(MY_INTERNAL_VAR)

# String choices for cmake-gui dropdown
set(LOG_LEVEL "INFO" CACHE STRING "Logging level")
set_property(CACHE LOG_LEVEL PROPERTY STRINGS "DEBUG;INFO;WARN;ERROR")
```

### Environment Variables

```cmake
# Read env variable
$ENV{HOME}
$ENV{PATH}

# Set env variable (only for CMake's runtime, not the shell)
set(ENV{MY_VAR} "value")

# Check if set
if(DEFINED ENV{CI})
    message(STATUS "Running in CI")
endif()
```

### String Operations

```cmake
set(str "Hello, World!")

string(LENGTH "${str}" len)                    # len = 13
string(TOUPPER "${str}" upper)                 # HELLO, WORLD!
string(TOLOWER "${str}" lower)                 # hello, world!
string(REPLACE "World" "CMake" result "${str}") # Hello, CMake!
string(SUBSTRING "${str}" 7 5 sub)             # World
string(FIND "${str}" "World" idx)              # idx = 7
string(REGEX MATCH    "[0-9]+" num "${str}")   # first match
string(REGEX MATCHALL "[a-z]+" words "${str}") # all matches
string(REGEX REPLACE  "[aeiou]" "*" res "${str}")
string(STRIP "  hello  " stripped)             # "hello"
string(PREPEND str ">>> ")
string(APPEND str " <<<")

# Configure a header file with CMake variables
configure_file(config.h.in config.h @ONLY)
# In config.h.in: #define VERSION "@PROJECT_VERSION@"
```

---

## 8. Control Flow

### if / elseif / else

```cmake
# Basic conditions
if(MY_VAR)                    # True if non-empty, non-zero, not "FALSE/OFF/NO/0/NOTFOUND"
if(NOT MY_VAR)
if(MY_VAR STREQUAL "hello")   # String equal
if(MY_VAR STRGREATER "abc")   # String greater
if(MY_VAR STRLESS "abc")
if(MY_VAR MATCHES "^[0-9]+$") # Regex match

# Numeric comparison
if(NUM EQUAL 42)
if(NUM GREATER 10)
if(NUM LESS 100)
if(NUM GREATER_EQUAL 5)
if(NUM LESS_EQUAL 99)

# Version comparison
if(CMAKE_VERSION VERSION_GREATER_EQUAL "3.20")
if(BOOST_VERSION VERSION_LESS "1.75")

# Existence checks
if(DEFINED MY_VAR)            # Variable is defined
if(DEFINED CACHE{MY_VAR})     # Cache variable is defined
if(TARGET mylib)              # Target exists
if(EXISTS "/path/to/file")    # File or directory exists
if(IS_DIRECTORY "/some/dir")
if(IS_ABSOLUTE "/some/path")
if(COMMAND find_package)      # CMake command exists

# Boolean operators
if(A AND B)
if(A OR B)
if(NOT A)
if((A OR B) AND C)

# Platform checks
if(WIN32)       # Windows (including 64-bit)
if(UNIX)        # Linux, macOS, and other Unix-like
if(APPLE)       # macOS
if(LINUX)       # Linux (CMake 3.25+)
if(MSVC)        # Microsoft Visual C++ compiler
if(CMAKE_COMPILER_IS_GNUCXX)  # GCC
```

### foreach

```cmake
# Over a list
foreach(item alpha beta gamma)
    message(STATUS "Item: ${item}")
endforeach()

# Over a variable list
set(FRUITS apple banana cherry)
foreach(fruit IN LISTS FRUITS)
    message(STATUS "Fruit: ${fruit}")
endforeach()

# C-style range
foreach(i RANGE 0 9)        # 0, 1, 2, ..., 9
    message(STATUS "i = ${i}")
endforeach()

foreach(i RANGE 0 20 5)     # 0, 5, 10, 15, 20 (with step)
    message(STATUS "i = ${i}")
endforeach()

# Zip multiple lists (CMake 3.17+)
set(NAMES Alice Bob)
set(AGES  30    25)
foreach(name age IN ZIP_LISTS NAMES AGES)
    message(STATUS "${name} is ${age}")
endforeach()

# break and continue
foreach(item IN LISTS MY_LIST)
    if(item STREQUAL "skip")
        continue()
    endif()
    if(item STREQUAL "stop")
        break()
    endif()
    message(STATUS "${item}")
endforeach()
```

### while

```cmake
set(counter 0)
while(counter LESS 5)
    message(STATUS "counter = ${counter}")
    math(EXPR counter "${counter} + 1")
endwhile()
```

### Math

```cmake
math(EXPR result "2 + 2")           # result = "4"
math(EXPR result "(10 * 3) / 5")    # result = "6"
math(EXPR result "1 << 4")          # result = "16" (bit shift)
math(EXPR result "0xFF & 0x0F")     # Bitwise AND
math(EXPR hex "255" OUTPUT_FORMAT HEXADECIMAL)  # hex = "0xff"
```

---

## 9. Functions & Macros

### Functions (preferred — have their own scope)

```cmake
# Define a function
function(my_function arg1 arg2)
    message(STATUS "arg1 = ${arg1}")
    message(STATUS "arg2 = ${arg2}")

    # ARGC = total argument count
    # ARGV = all arguments as a list
    # ARGN = arguments beyond declared params
    message(STATUS "Total args: ${ARGC}")

    # Return value via PARENT_SCOPE
    set(RESULT "${arg1}_${arg2}" PARENT_SCOPE)
endfunction()

# Call it
my_function("hello" "world")
message(STATUS "Result: ${RESULT}")

# Variadic function
function(print_all)
    foreach(arg IN LISTS ARGN)
        message(STATUS "  ${arg}")
    endforeach()
endfunction()

print_all(one two three four)
```

### Macros (text substitution — no scope)

```cmake
# Macro: like a function but variables leak into caller's scope
macro(my_macro name)
    set(${name}_processed TRUE)   # Sets in caller's scope
    message(STATUS "Processing ${name}")
endmacro()

my_macro(MyTarget)
# MyTarget_processed is now TRUE in this scope
```

### cmake_parse_arguments (named parameters)

```cmake
function(create_target)
    # Define expected arguments
    cmake_parse_arguments(
        PARSE_ARGV 0   # Start parsing from argument 0
        ARG            # Prefix for result variables
        "SHARED;STATIC"                     # Options (flags, no value)
        "NAME;OUTPUT_DIR"                   # Single-value keywords
        "SOURCES;HEADERS;DEPENDENCIES"      # Multi-value keywords
    )

    # Access parsed values
    message(STATUS "Name: ${ARG_NAME}")
    message(STATUS "Sources: ${ARG_SOURCES}")
    message(STATUS "Is shared: ${ARG_SHARED}")

    if(ARG_SHARED)
        add_library(${ARG_NAME} SHARED ${ARG_SOURCES})
    else()
        add_library(${ARG_NAME} STATIC ${ARG_SOURCES})
    endif()

    if(ARG_DEPENDENCIES)
        target_link_libraries(${ARG_NAME} PRIVATE ${ARG_DEPENDENCIES})
    endif()

    # ARG_UNPARSED_ARGUMENTS → anything not matched
    if(ARG_UNPARSED_ARGUMENTS)
        message(WARNING "Unknown args: ${ARG_UNPARSED_ARGUMENTS}")
    endif()
endfunction()

# Usage
create_target(
    NAME       mylib
    SHARED
    SOURCES    lib.cpp utils.cpp
    HEADERS    lib.h utils.h
    DEPENDENCIES fmt::fmt spdlog::spdlog
)
```

### Useful Helper Patterns

```cmake
# Add a library + tests + set common properties in one call
function(add_my_library name)
    cmake_parse_arguments(PARSE_ARGV 1 ARG "" "" "SOURCES;TESTS;DEPS")

    add_library(${name} STATIC ${ARG_SOURCES})
    add_library(MyProject::${name} ALIAS ${name})

    target_include_directories(${name} PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    )

    target_compile_features(${name} PUBLIC cxx_std_17)

    if(ARG_DEPS)
        target_link_libraries(${name} PUBLIC ${ARG_DEPS})
    endif()

    if(ARG_TESTS AND BUILD_TESTING)
        foreach(test_src IN LISTS ARG_TESTS)
            get_filename_component(test_name ${test_src} NAME_WE)
            add_executable(${test_name} ${test_src})
            target_link_libraries(${test_name} PRIVATE ${name} GTest::gtest_main)
            add_test(NAME ${test_name} COMMAND ${test_name})
        endforeach()
    endif()
endfunction()
```

---

## 10. Finding External Dependencies

### find_package

```cmake
# Basic usage
find_package(OpenSSL REQUIRED)
find_package(Boost   REQUIRED COMPONENTS system filesystem)
find_package(fmt     REQUIRED)
find_package(ZLIB    QUIET)    # QUIET: don't print anything if not found

# Check if found
if(ZLIB_FOUND)
    target_link_libraries(myapp PRIVATE ZLIB::ZLIB)
endif()

# Version constraints
find_package(OpenSSL 1.1.0 REQUIRED)   # At least 1.1.0
find_package(Boost   1.74  EXACT)      # Exactly 1.74

# Use modern namespaced targets (preferred)
target_link_libraries(myapp PRIVATE
    OpenSSL::SSL
    OpenSSL::Crypto
    Boost::system
    Boost::filesystem
    fmt::fmt
    ZLIB::ZLIB
)

# Common packages and their targets
# Package            | CMake Target
# -------------------|------------------------------
# OpenSSL            | OpenSSL::SSL, OpenSSL::Crypto
# Boost              | Boost::<component>
# ZLIB               | ZLIB::ZLIB
# fmt                | fmt::fmt
# spdlog             | spdlog::spdlog
# GTest              | GTest::gtest, GTest::gtest_main
# Threads            | Threads::Threads
# OpenGL             | OpenGL::GL
# CURL               | CURL::libcurl
# SQLite3            | SQLite::SQLite3
# nlohmann_json      | nlohmann_json::nlohmann_json
```

### Writing a Custom Find Module

When a library doesn't provide CMake config files, write a `FindMyLib.cmake`:

```cmake
# cmake/FindMyLib.cmake

# Search for header
find_path(MYLIB_INCLUDE_DIR
    NAMES mylib.h
    PATHS /usr/include /usr/local/include
    PATH_SUFFIXES mylib
)

# Search for library
find_library(MYLIB_LIBRARY
    NAMES mylib
    PATHS /usr/lib /usr/local/lib
)

# Version detection (optional)
if(MYLIB_INCLUDE_DIR AND EXISTS "${MYLIB_INCLUDE_DIR}/mylib_version.h")
    file(STRINGS "${MYLIB_INCLUDE_DIR}/mylib_version.h" version_line
         REGEX "^#define MYLIB_VERSION ")
    string(REGEX REPLACE ".*\"([^\"]+)\".*" "\\1" MYLIB_VERSION "${version_line}")
endif()

# Standard handling (sets MYLIB_FOUND, handles REQUIRED/QUIET)
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(MyLib
    REQUIRED_VARS MYLIB_LIBRARY MYLIB_INCLUDE_DIR
    VERSION_VAR   MYLIB_VERSION
)

# Create an imported target (modern style)
if(MYLIB_FOUND AND NOT TARGET MyLib::MyLib)
    add_library(MyLib::MyLib UNKNOWN IMPORTED)
    set_target_properties(MyLib::MyLib PROPERTIES
        IMPORTED_LOCATION             "${MYLIB_LIBRARY}"
        INTERFACE_INCLUDE_DIRECTORIES "${MYLIB_INCLUDE_DIR}"
    )
endif()

mark_as_advanced(MYLIB_INCLUDE_DIR MYLIB_LIBRARY)
```

```cmake
# In root CMakeLists.txt — tell CMake where to find custom modules
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
find_package(MyLib REQUIRED)
```

### pkg-config

```cmake
find_package(PkgConfig REQUIRED)

# Check for a package
pkg_check_modules(GTK3 REQUIRED gtk+-3.0)
pkg_check_modules(CAIRO cairo)

# Use the results (old style)
target_include_directories(myapp PRIVATE ${GTK3_INCLUDE_DIRS})
target_link_libraries(myapp PRIVATE ${GTK3_LIBRARIES})
target_compile_options(myapp PRIVATE ${GTK3_CFLAGS_OTHER})

# Import as a target (CMake 3.6+, cleaner)
pkg_check_modules(GTK3 REQUIRED IMPORTED_TARGET gtk+-3.0)
target_link_libraries(myapp PRIVATE PkgConfig::GTK3)
```

---

## 11. FetchContent & CPM (Dependency Management)

### FetchContent (Built-in, CMake 3.11+)

Downloads and integrates dependencies at configure time.

```cmake
include(FetchContent)

# Declare what to fetch
FetchContent_Declare(
    fmt
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG        10.2.1
    GIT_SHALLOW    TRUE          # Only download the tip commit
)

FetchContent_Declare(
    nlohmann_json
    GIT_REPOSITORY https://github.com/nlohmann/json.git
    GIT_TAG        v3.11.3
    GIT_SHALLOW    TRUE
)

# From a URL (no git needed)
FetchContent_Declare(
    catch2
    URL      https://github.com/catchorg/Catch2/archive/refs/tags/v3.4.0.tar.gz
    URL_HASH SHA256=122928b814b75717316c71af69bd2b43387643ba076a6ec16e7882bfb2dfacbb
)

# Make all declared dependencies available
FetchContent_MakeAvailable(fmt nlohmann_json catch2)

# Now use them like any other target
target_link_libraries(myapp PRIVATE
    fmt::fmt
    nlohmann_json::nlohmann_json
)
```

### FetchContent Options

```cmake
# Disable tests/examples from the fetched project
set(FMT_TEST OFF CACHE BOOL "" FORCE)
set(JSON_BuildTests OFF CACHE INTERNAL "")
set(CATCH_INSTALL_EXTRAS OFF)

FetchContent_Declare(fmt GIT_REPOSITORY ... GIT_TAG ...)
FetchContent_MakeAvailable(fmt)

# Check if already available (avoids re-downloading)
FetchContent_GetProperties(fmt)
if(NOT fmt_POPULATED)
    FetchContent_Populate(fmt)
    add_subdirectory(${fmt_SOURCE_DIR} ${fmt_BINARY_DIR} EXCLUDE_FROM_ALL)
endif()
```

### CPM.cmake (Third-party, simpler syntax)

```cmake
# First, download CPM.cmake:
# cmake/CPM.cmake from https://github.com/cpm-cmake/CPM.cmake/releases

include(cmake/CPM.cmake)

CPMAddPackage("gh:fmtlib/fmt#10.2.1")
CPMAddPackage("gh:nlohmann/json@3.11.3")
CPMAddPackage("gh:catchorg/Catch2@3.4.0")
CPMAddPackage(
    NAME spdlog
    GITHUB_REPOSITORY gabime/spdlog
    VERSION 1.12.0
    OPTIONS "SPDLOG_FMT_EXTERNAL ON"
)

target_link_libraries(myapp PRIVATE fmt::fmt spdlog::spdlog)
```

---

## 12. Installing & Packaging

### Installing Targets

```cmake
include(GNUInstallDirs)  # Provides standard directory variables

# Install targets
install(TARGETS myapp mylib
    RUNTIME  DESTINATION ${CMAKE_INSTALL_BINDIR}      # Executables → bin/
    LIBRARY  DESTINATION ${CMAKE_INSTALL_LIBDIR}      # Shared libs → lib/
    ARCHIVE  DESTINATION ${CMAKE_INSTALL_LIBDIR}      # Static libs → lib/
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}  # For export
)

# Install headers
install(DIRECTORY include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp"
)

# Install specific files
install(FILES README.md LICENSE
    DESTINATION ${CMAKE_INSTALL_DOCDIR}
)

# Install programs (sets executable bit)
install(PROGRAMS scripts/run.sh
    DESTINATION ${CMAKE_INSTALL_BINDIR}
)

# Run install
cmake --install build --prefix /usr/local
cmake --install build --prefix /usr/local --component runtime  # Selective
```

### Export & Config Files (for other projects to use your library)

```cmake
# In your library's CMakeLists.txt:

install(TARGETS mylib
    EXPORT MyProjectTargets
    ...
)

# Install the export file
install(EXPORT MyProjectTargets
    FILE        MyProjectTargets.cmake
    NAMESPACE   MyProject::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
)

# Create MyProjectConfig.cmake
include(CMakePackageConfigHelpers)

configure_package_config_file(
    cmake/MyProjectConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
)

write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion    # AnyNewerVersion, SameMajorVersion, SameMinorVersion, ExactVersion
)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
)
```

```cmake
# cmake/MyProjectConfig.cmake.in
@PACKAGE_INIT@

include("${CMAKE_CURRENT_LIST_DIR}/MyProjectTargets.cmake")
check_required_components(MyProject)
```

### CPack (Binary Packaging)

```cmake
# At the end of your root CMakeLists.txt
set(CPACK_PACKAGE_NAME "MyProject")
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "My great project")
set(CPACK_PACKAGE_VENDOR "My Company")
set(CPACK_PACKAGE_CONTACT "email@example.com")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/LICENSE")

# Generators
set(CPACK_GENERATOR "TGZ;ZIP")           # tarballs
set(CPACK_GENERATOR "DEB")               # Debian package
set(CPACK_GENERATOR "RPM")               # RPM package
set(CPACK_GENERATOR "NSIS")              # Windows installer
set(CPACK_GENERATOR "productbuild")      # macOS package

# DEB specifics
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libc6 (>= 2.17)")
set(CPACK_DEBIAN_PACKAGE_SECTION "devel")

include(CPack)
```

```bash
cd build
cpack                        # Create packages
cpack -G TGZ                 # Specific generator
cpack -G DEB --config CPackConfig.cmake
```

---

## 13. Testing with CTest

### Basic CTest Setup

```cmake
# In root CMakeLists.txt
option(BUILD_TESTING "Build tests" ON)
if(BUILD_TESTING)
    enable_testing()
    add_subdirectory(tests)
endif()
```

```cmake
# tests/CMakeLists.txt

# Simple executable test
add_executable(test_math test_math.cpp)
target_link_libraries(test_math PRIVATE mylib)
add_test(NAME MathTest COMMAND test_math)

# Test with arguments
add_test(NAME ParseTest COMMAND myapp --parse input.txt)

# Test with expected pass/fail
add_test(NAME FailTest COMMAND myapp --bad-arg)
set_tests_properties(FailTest PROPERTIES WILL_FAIL TRUE)

# Test with regex to check output
add_test(NAME VersionTest COMMAND myapp --version)
set_tests_properties(VersionTest PROPERTIES
    PASS_REGULAR_EXPRESSION "v[0-9]+\\.[0-9]+"
)

# Test timeout
set_tests_properties(SlowTest PROPERTIES TIMEOUT 30)

# Test labels (for filtering)
set_tests_properties(MathTest PROPERTIES LABELS "unit;fast")
set_tests_properties(IntegrationTest PROPERTIES LABELS "integration;slow")
```

### Google Test Integration

```cmake
include(FetchContent)
FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG        v1.14.0
    GIT_SHALLOW    TRUE
)
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)  # Windows
FetchContent_MakeAvailable(googletest)

include(GoogleTest)

add_executable(test_mylib test_mylib.cpp)
target_link_libraries(test_mylib PRIVATE mylib GTest::gtest_main)

# Auto-discover all TEST() and TEST_F() tests
gtest_discover_tests(test_mylib)
```

```cpp
// test_mylib.cpp
#include <gtest/gtest.h>
#include "mylib.h"

TEST(MathTest, Addition) {
    EXPECT_EQ(add(2, 3), 5);
    EXPECT_EQ(add(-1, 1), 0);
}

TEST(MathTest, Division) {
    EXPECT_DOUBLE_EQ(divide(10.0, 4.0), 2.5);
    EXPECT_THROW(divide(1.0, 0.0), std::invalid_argument);
}
```

### Catch2 Integration

```cmake
FetchContent_Declare(
    Catch2
    GIT_REPOSITORY https://github.com/catchorg/Catch2.git
    GIT_TAG        v3.4.0
    GIT_SHALLOW    TRUE
)
FetchContent_MakeAvailable(Catch2)

add_executable(test_mylib test_mylib.cpp)
target_link_libraries(test_mylib PRIVATE mylib Catch2::Catch2WithMain)

include(Catch)
catch_discover_tests(test_mylib)
```

### Running Tests

```bash
cd build
ctest                          # Run all tests
ctest -V                       # Verbose output
ctest -j8                      # Run 8 tests in parallel
ctest --output-on-failure      # Print output only on failure
ctest -R "Math"                # Run tests matching regex
ctest -E "Slow"                # Exclude tests matching regex
ctest -L "unit"                # Run tests with label "unit"
ctest --rerun-failed           # Re-run only failed tests
ctest -C Release               # Specify configuration (multi-config)
ctest --test-dir build         # Specify build dir
```

---

## 14. Generator Expressions

Generator expressions are evaluated at **build time** (not configure time). They're written as `$<...>` and are useful for per-configuration logic.

### Conditional Expressions

```cmake
# $<condition:value> — value if condition is true, empty otherwise
$<$<CONFIG:Debug>:DEBUG_MODE>           # Set DEBUG_MODE only in Debug builds
$<$<PLATFORM_ID:Windows>:WIN32>         # Only on Windows
$<$<CXX_COMPILER_ID:MSVC>:/W4>         # Only with MSVC
$<$<BOOL:${MY_VAR}>:some_value>         # Based on CMake variable

# $<IF:condition,true_val,false_val>
$<IF:$<CONFIG:Debug>,debug_lib,release_lib>
```

### Target & Property Expressions

```cmake
$<TARGET_FILE:myapp>             # Full path to myapp's binary
$<TARGET_FILE_DIR:myapp>         # Directory containing myapp
$<TARGET_FILE_NAME:myapp>        # Just the filename
$<TARGET_LINKER_FILE:mylib>      # Linker file (import lib on Windows)

$<TARGET_PROPERTY:mylib,INCLUDE_DIRECTORIES>   # A target's property
$<TARGET_EXISTS:mylib>           # 1 if target exists
```

### Configuration Expressions

```cmake
$<CONFIG>                        # Current config name (Debug, Release...)
$<CONFIG:Debug>                  # 1 if Debug build
$<CONFIG:Debug,Release>          # 1 if Debug OR Release

# Common pattern: different libraries for debug/release
target_link_libraries(myapp PRIVATE
    $<$<CONFIG:Debug>:debug_lib>
    $<$<CONFIG:Release>:release_lib>
)
```

### Build vs Install Interface

```cmake
# The most important generator expression pattern:
target_include_directories(mylib PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>   # When building
    $<INSTALL_INTERFACE:include>                              # After installing
)
```

### Other Useful Expressions

```cmake
$<COMPILE_LANGUAGE:CXX>          # True when compiling C++ files
$<LINK_LANGUAGE:CXX>             # True when linking C++ targets

# Join a list with separator
$<JOIN:${MY_LIST},:>             # Joins with colon

# Remove duplicates
$<REMOVE_DUPLICATES:a;b;a;c>     # → a;b;c

# Shell-compatible path separator
$<PATH_JOIN:a,b,c>               # → a/b/c or a\b\c
```

---

## 15. Toolchains & Cross-Compilation

### Specifying Compilers

```bash
# Via environment variables (before cmake)
CC=clang CXX=clang++ cmake ..

# Via CMake variables
cmake -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ ..

# Note: cannot change compilers after first configure without deleting build dir
```

### Toolchain Files

For cross-compilation, create a toolchain file and pass it with `-DCMAKE_TOOLCHAIN_FILE=`.

```cmake
# toolchains/arm-linux-gnueabihf.cmake

# Target system
set(CMAKE_SYSTEM_NAME    Linux)
set(CMAKE_SYSTEM_PROCESSOR arm)

# Cross compilers
set(CMAKE_C_COMPILER   arm-linux-gnueabihf-gcc)
set(CMAKE_CXX_COMPILER arm-linux-gnueabihf-g++)
set(CMAKE_STRIP        arm-linux-gnueabihf-strip)

# Search paths — where to find headers/libs for the target
set(CMAKE_FIND_ROOT_PATH /usr/arm-linux-gnueabihf)

# Search rules: find programs in host, headers/libs in target
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
```

```bash
cmake -DCMAKE_TOOLCHAIN_FILE=toolchains/arm-linux-gnueabihf.cmake -S . -B build-arm
cmake --build build-arm
```

### Vcpkg Integration

```bash
# Install vcpkg
git clone https://github.com/microsoft/vcpkg.git
./vcpkg/bootstrap-vcpkg.sh

# Use with CMake
cmake -DCMAKE_TOOLCHAIN_FILE=/path/to/vcpkg/scripts/buildsystems/vcpkg.cmake ..
```

```cmake
# vcpkg.json (manifest mode — preferred)
{
    "name": "my-project",
    "version": "1.0.0",
    "dependencies": [
        "fmt",
        "boost-system",
        "openssl"
    ]
}
```

### Compiler Detection & Platform Guards

```cmake
# Compiler ID strings: GNU, Clang, AppleClang, MSVC, Intel, IntelLLVM
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    target_compile_options(myapp PRIVATE -Wall -Wextra)
elseif(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(myapp PRIVATE -Weverything -Wno-c++98-compat)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_compile_options(myapp PRIVATE /W4 /WX)
endif()

# Architecture detection
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    message(STATUS "64-bit build")
else()
    message(STATUS "32-bit build")
endif()
```

---

## 16. CMake Presets

Presets (CMake 3.19+) let you define reusable configurations in `CMakePresets.json`.

```json
{
    "version": 6,
    "cmakeMinimumRequired": { "major": 3, "minor": 20, "patch": 0 },

    "configurePresets": [
        {
            "name": "base",
            "hidden": true,
            "generator": "Ninja",
            "binaryDir": "${sourceDir}/build/${presetName}",
            "installDir": "${sourceDir}/install/${presetName}"
        },
        {
            "name": "debug",
            "inherits": "base",
            "displayName": "Debug Build",
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "Debug",
                "BUILD_TESTING": "ON"
            }
        },
        {
            "name": "release",
            "inherits": "base",
            "displayName": "Release Build",
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "Release",
                "BUILD_TESTING": "OFF"
            }
        },
        {
            "name": "clang",
            "inherits": "debug",
            "displayName": "Clang Debug",
            "cacheVariables": {
                "CMAKE_C_COMPILER":   "clang",
                "CMAKE_CXX_COMPILER": "clang++"
            }
        },
        {
            "name": "cross-arm",
            "inherits": "release",
            "displayName": "ARM Cross-Compile",
            "toolchainFile": "toolchains/arm-linux-gnueabihf.cmake"
        }
    ],

    "buildPresets": [
        {
            "name": "debug",
            "configurePreset": "debug",
            "jobs": 8
        },
        {
            "name": "release",
            "configurePreset": "release",
            "jobs": 8
        }
    ],

    "testPresets": [
        {
            "name": "debug",
            "configurePreset": "debug",
            "output": { "outputOnFailure": true },
            "execution": { "jobs": 4, "stopOnFailure": false }
        }
    ]
}
```

```bash
# List available presets
cmake --list-presets

# Configure with a preset
cmake --preset debug
cmake --preset release

# Build with a preset
cmake --build --preset debug

# Test with a preset
ctest --preset debug
```

---

## 17. Useful Built-in Modules

```cmake
# Include a module
include(ModuleName)

# GNUInstallDirs — standard installation directories
include(GNUInstallDirs)
# Provides: CMAKE_INSTALL_BINDIR, LIBDIR, INCLUDEDIR, DATADIR, DOCDIR, etc.

# CMakePrintHelpers — debugging
include(CMakePrintHelpers)
cmake_print_variables(MY_VAR CMAKE_BUILD_TYPE)         # Print variable values
cmake_print_properties(TARGETS mylib PROPERTIES INCLUDE_DIRECTORIES)

# CheckCXXCompilerFlag — test if a flag is supported
include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("-std=c++20" COMPILER_SUPPORTS_CXX20)
if(COMPILER_SUPPORTS_CXX20)
    target_compile_options(myapp PRIVATE -std=c++20)
endif()

# CheckCXXSourceCompiles — test if code compiles
include(CheckCXXSourceCompiles)
check_cxx_source_compiles("
    #include <filesystem>
    int main() { std::filesystem::path p; return 0; }
" HAS_FILESYSTEM)

# CheckIncludeFileCXX
include(CheckIncludeFileCXX)
check_include_file_cxx("optional" HAS_OPTIONAL)

# WriteCompilerDetectionHeader
include(WriteCompilerDetectionHeader)

# CTest & CDash integration
include(CTest)

# FetchContent
include(FetchContent)

# CMakePackageConfigHelpers
include(CMakePackageConfigHelpers)

# ExternalProject — build at build time (not configure time like FetchContent)
include(ExternalProject)
ExternalProject_Add(
    myexternal
    GIT_REPOSITORY https://github.com/example/lib.git
    GIT_TAG        main
    CMAKE_ARGS     -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
)
```

### Generating Files

```cmake
# configure_file — substitute @VAR@ or ${VAR} in a template
configure_file(
    ${CMAKE_SOURCE_DIR}/include/version.h.in
    ${CMAKE_BINARY_DIR}/include/version.h
    @ONLY    # Only replace @VAR@, not ${VAR}
)
```

```c
// include/version.h.in
#pragma once
#define VERSION_MAJOR @PROJECT_VERSION_MAJOR@
#define VERSION_MINOR @PROJECT_VERSION_MINOR@
#define VERSION_PATCH @PROJECT_VERSION_PATCH@
#define VERSION_STRING "@PROJECT_VERSION@"
#define BUILD_TYPE "@CMAKE_BUILD_TYPE@"
```

```cmake
# file(GENERATE) — generate file using generator expressions (evaluated at gen time)
file(GENERATE
    OUTPUT  ${CMAKE_BINARY_DIR}/run_$<CONFIG>.sh
    CONTENT "#!/bin/bash\n$<TARGET_FILE:myapp> \"$@\"\n"
)

# add_custom_command — run a command as part of the build
add_custom_command(
    OUTPUT  ${CMAKE_BINARY_DIR}/generated.cpp
    COMMAND python3 ${CMAKE_SOURCE_DIR}/scripts/codegen.py
            --output ${CMAKE_BINARY_DIR}/generated.cpp
    DEPENDS ${CMAKE_SOURCE_DIR}/scripts/codegen.py
            ${CMAKE_SOURCE_DIR}/schema.json
    COMMENT "Generating source code..."
)
add_executable(myapp main.cpp ${CMAKE_BINARY_DIR}/generated.cpp)

# add_custom_target — a target that always runs
add_custom_target(format
    COMMAND clang-format -i ${ALL_SOURCES}
    COMMENT "Formatting source code..."
)
```

---

## 18. Real-World Project Layout

### Full Project Example

```
MyProject/
├── CMakeLists.txt
├── CMakePresets.json
├── vcpkg.json
├── cmake/
│   ├── CPM.cmake
│   ├── MyProjectConfig.cmake.in
│   └── FindMyLib.cmake
├── include/
│   └── myproject/
│       ├── core.h
│       └── utils.h
├── src/
│   ├── CMakeLists.txt
│   ├── core.cpp
│   └── utils.cpp
├── app/
│   ├── CMakeLists.txt
│   └── main.cpp
├── tests/
│   ├── CMakeLists.txt
│   ├── test_core.cpp
│   └── test_utils.cpp
└── docs/
    └── CMakeLists.txt
```

### Root CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.20)

project(
    MyProject
    VERSION     1.2.0
    DESCRIPTION "A well-structured CMake project"
    LANGUAGES   CXX
)

# ── Guard against in-source builds ──────────────────────────────────────
if(CMAKE_SOURCE_DIR STREQUAL CMAKE_BINARY_DIR)
    message(FATAL_ERROR
        "In-source builds are not allowed.\n"
        "Run: cmake -S . -B build")
endif()

# ── Project-wide settings ────────────────────────────────────────────────
set(CMAKE_CXX_STANDARD          20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS        OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)   # For clangd/LSP

# Put all binaries in predictable locations
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# ── Options ──────────────────────────────────────────────────────────────
option(BUILD_TESTING  "Build test suite"     ON)
option(BUILD_DOCS     "Build documentation"  OFF)
option(ENABLE_ASAN    "Enable AddressSanitizer" OFF)
option(ENABLE_TSAN    "Enable ThreadSanitizer"  OFF)

# ── Sanitizers ───────────────────────────────────────────────────────────
if(ENABLE_ASAN)
    add_compile_options(-fsanitize=address,undefined -fno-omit-frame-pointer)
    add_link_options(-fsanitize=address,undefined)
endif()
if(ENABLE_TSAN)
    add_compile_options(-fsanitize=thread)
    add_link_options(-fsanitize=thread)
endif()

# ── Module path ──────────────────────────────────────────────────────────
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

# ── Dependencies (via CPM) ───────────────────────────────────────────────
include(cmake/CPM.cmake)

CPMAddPackage("gh:fmtlib/fmt#10.2.1")
CPMAddPackage("gh:gabime/spdlog@1.12.0")

set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
CPMAddPackage("gh:google/googletest@1.14.0")

# ── Subdirectories ───────────────────────────────────────────────────────
add_subdirectory(src)
add_subdirectory(app)

if(BUILD_TESTING)
    enable_testing()
    add_subdirectory(tests)
endif()

if(BUILD_DOCS)
    add_subdirectory(docs)
endif()

# ── Install & Packaging ──────────────────────────────────────────────────
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

configure_package_config_file(
    cmake/MyProjectConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
)

include(CPack)
```

### src/CMakeLists.txt

```cmake
add_library(myproject-core STATIC
    core.cpp
    utils.cpp
)

# Modern alias — use MyProject::core everywhere else
add_library(MyProject::core ALIAS myproject-core)

target_include_directories(myproject-core PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

target_link_libraries(myproject-core
    PUBLIC  fmt::fmt
    PRIVATE spdlog::spdlog
)

target_compile_features(myproject-core PUBLIC cxx_std_20)

install(TARGETS myproject-core
    EXPORT  MyProjectTargets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
```

### app/CMakeLists.txt

```cmake
add_executable(myapp main.cpp)

target_link_libraries(myapp PRIVATE MyProject::core)

set_target_properties(myapp PROPERTIES OUTPUT_NAME "myproject")

install(TARGETS myapp RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
```

### tests/CMakeLists.txt

```cmake
include(GoogleTest)

function(add_unit_test name)
    add_executable(${name} ${name}.cpp)
    target_link_libraries(${name} PRIVATE
        MyProject::core
        GTest::gtest_main
    )
    gtest_discover_tests(${name}
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    )
endfunction()

add_unit_test(test_core)
add_unit_test(test_utils)
```

---

## 19. Best Practices & Common Pitfalls

### Do This ✅

```cmake
# Use target_* commands — never set global variables for build settings
target_include_directories(mylib PUBLIC include/)     # ✅
include_directories(include/)                         # ❌ global, pollutes everything

target_compile_options(mylib PRIVATE -Wall)           # ✅
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")       # ❌ global, hard to manage

target_link_libraries(myapp PRIVATE mylib)            # ✅ with PRIVATE/PUBLIC/INTERFACE
target_link_libraries(myapp mylib)                    # ❌ missing visibility keyword

# Use namespaced aliases
add_library(MyProject::mylib ALIAS mylib)             # ✅

# Use modern find_package targets
target_link_libraries(myapp PRIVATE OpenSSL::SSL)     # ✅
target_link_libraries(myapp PRIVATE ${OPENSSL_LIBRARIES})  # ❌ old style

# Set CXX standard via target_compile_features
target_compile_features(mylib PUBLIC cxx_std_17)      # ✅
set(CMAKE_CXX_STANDARD 17)                            # ⚠️  ok at top-level, but prefer features

# Out-of-source builds — always!
cmake -S . -B build                                   # ✅
cmake .                                               # ❌ in-source build

# Use generator expressions for config-dependent logic
target_compile_definitions(mylib PRIVATE
    $<$<CONFIG:Debug>:MY_DEBUG>)                      # ✅
if(CMAKE_BUILD_TYPE STREQUAL Debug)                   # ❌ doesn't work for multi-config
    target_compile_definitions(mylib PRIVATE MY_DEBUG)
endif()
```

### Don't Do This ❌

```cmake
# Don't use file(GLOB) for source files — CMake won't reconfigure on new files
file(GLOB SOURCES "src/*.cpp")                        # ❌
# Use explicit list or CONFIGURE_DEPENDS (slower but correct)
file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS "src/*.cpp")  # ⚠️ acceptable

# Don't set CMAKE_* variables for one target — use target_* instead
set(CMAKE_CXX_FLAGS "-O3")                            # ❌
set(CMAKE_INCLUDE_CURRENT_DIR ON)                     # ❌

# Don't use add_compile_options() for target-specific flags
add_compile_options(-Wall)                            # ❌ affects all targets
target_compile_options(myapp PRIVATE -Wall)           # ✅

# Don't link libraries without visibility keywords (before CMake 3.13 this was a warning)
target_link_libraries(myapp mylib)                    # ❌ use PRIVATE/PUBLIC/INTERFACE

# Don't use PUBLIC for private implementation details
target_link_libraries(mylib PUBLIC spdlog::spdlog)   # ❌ if spdlog is impl detail
target_link_libraries(mylib PRIVATE spdlog::spdlog)  # ✅

# Don't use string comparison for build type in multi-config generators
if(CMAKE_BUILD_TYPE STREQUAL "Debug")                 # ❌ empty in multi-config
    # Use generator expressions instead

# Don't hardcode paths
set(INCLUDES "/usr/local/include")                    # ❌
find_package(MyLib REQUIRED)                          # ✅ let CMake find it
```

### Debugging CMake

```cmake
# Print messages
message(STATUS "Configuring for ${CMAKE_BUILD_TYPE}")     # [-- message]
message(WARNING "Missing optional component")              # [CMake Warning]
message(SEND_ERROR "Non-fatal error — config continues")  # [CMake Error]
message(FATAL_ERROR "Stop everything now!")                # Aborts CMake
message(VERBOSE "Extra detail")                            # Only with --log-level=VERBOSE
message(DEBUG "Debug info")                                # Only with --log-level=DEBUG

# Print variable
message(STATUS "MY_VAR = ${MY_VAR}")
include(CMakePrintHelpers)
cmake_print_variables(MY_VAR CMAKE_CXX_COMPILER)

# Print all target properties
cmake_print_properties(TARGETS mylib PROPERTIES
    INCLUDE_DIRECTORIES
    LINK_LIBRARIES
    COMPILE_OPTIONS
)

# Get all cmake variables
get_cmake_property(ALL_VARS VARIABLES)
foreach(var IN LISTS ALL_VARS)
    message(STATUS "${var} = ${${var}}")
endforeach()
```

```bash
# Run cmake with extra output
cmake --log-level=VERBOSE ..
cmake --log-level=DEBUG   ..
cmake --trace             ..   # Show every command executed
cmake --trace-expand      ..   # Show with variables expanded
cmake --trace-source=CMakeLists.txt ..   # Only trace specific file
```

---

## 20. Quick Reference Card

### Most Common Commands

| Command | Purpose |
|---------|---------|
| `cmake_minimum_required(VERSION x.y)` | Set minimum CMake version |
| `project(Name VERSION x.y LANGUAGES CXX)` | Declare project |
| `add_executable(name src.cpp ...)` | Create an executable |
| `add_library(name [STATIC\|SHARED] src.cpp)` | Create a library |
| `target_link_libraries(name PRIVATE dep)` | Link a dependency |
| `target_include_directories(name PUBLIC dir)` | Add include path |
| `target_compile_options(name PRIVATE flags)` | Add compiler flags |
| `target_compile_definitions(name PRIVATE DEF)` | Add preprocessor define |
| `target_compile_features(name PUBLIC cxx_std_17)` | Set C++ standard |
| `add_subdirectory(dir)` | Process a subdirectory |
| `find_package(Lib REQUIRED)` | Find an installed library |
| `set(VAR value)` | Set a variable |
| `option(VAR "desc" ON)` | Expose a boolean option |
| `message(STATUS "text")` | Print a message |
| `include(Module)` | Include a CMake module |
| `install(TARGETS ...)` | Define install rules |
| `add_test(NAME t COMMAND cmd)` | Register a test |
| `configure_file(in out)` | Generate a file from template |

### Command-Line Reference

```bash
# Configure
cmake -S <source> -B <build>
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTING=OFF
cmake --preset <name>

# Build
cmake --build <build>
cmake --build build --config Release    # Multi-config (MSVC, Xcode)
cmake --build build --target myapp     # Specific target
cmake --build build --parallel 8       # Parallel jobs
cmake --build build -- -j8             # Pass args to underlying build tool

# Install
cmake --install build
cmake --install build --prefix /opt/myapp
cmake --install build --component runtime

# Test
ctest --test-dir build
ctest --preset <name>
ctest -R <regex> -V

# Package
cd build && cpack
cpack -G DEB

# Open in cmake-gui
cmake-gui
ccmake .       # Curses TUI
```

### Visibility Quick Reference

```
target_link_libraries(A PRIVATE  B)  →  A uses B, B not exposed to A's users
target_link_libraries(A PUBLIC   B)  →  A uses B, B IS exposed to A's users
target_link_libraries(A INTERFACE B) →  A does NOT use B, B IS exposed to A's users

Same rules apply to:
  target_include_directories()
  target_compile_options()
  target_compile_definitions()
  target_compile_features()
```

### CMake Version Feature Timeline

| Version | Key Feature |
|---------|-------------|
| 3.0 | Target-based build system (`target_*` commands) |
| 3.1 | `target_compile_features`, C++11 support |
| 3.8 | C# and CUDA language support |
| 3.11 | `FetchContent`, `IMPORTED_TARGET` in pkg-config |
| 3.12 | `find_package` package registry, `OBJECT` library linking |
| 3.13 | Install without root, `target_link_options` |
| 3.14 | `FetchContent_MakeAvailable` |
| 3.16 | Unity builds, `target_precompile_headers` |
| 3.17 | `CMAKE_CUDA_STANDARD`, `foreach ZIP_LISTS` |
| 3.19 | CMake Presets (`CMakePresets.json`) |
| 3.20 | `cmake_path()`, C++23 standard support |
| 3.21 | `--install-dir` flag, `HIP` language |
| 3.24 | `FetchContent` with `find_package` integration |
| 3.25 | `LINUX` platform variable, block() scoping |
| 3.28 | C++20 module support |
