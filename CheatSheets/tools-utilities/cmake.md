# CMake: A Complete Progressive Tutorial

---

## 1. What & Why

CMake is a build system generator — it reads `CMakeLists.txt` configuration files and generates native build files for your platform and chosen toolchain: Makefiles on Linux/macOS, Visual Studio project files on Windows, Ninja build files, Xcode projects, and more. CMake itself doesn't compile your code. It writes the instructions that your native build tool uses to compile it.

Why use CMake over writing Makefiles directly? Because a hand-written Makefile for a real project becomes unmaintainable: it doesn't handle cross-platform compilation, finding dependencies, generating IDE project files, or managing complex build configurations. CMake abstracts all of this into a consistent, cross-platform description of what you want to build.

CMake is the de facto standard for C and C++ projects. Nearly every major open-source C/C++ library ships with CMake support, which means `find_package()` can locate and link them automatically.

---

## 2. Mental Model

CMake operates in three stages:

```
Stage 1: Configure
  CMake reads all CMakeLists.txt files, checks the compiler,
  finds libraries, evaluates conditions.
  Result: CMakeCache.txt (persists across reconfigures)

Stage 2: Generate
  CMake writes build files for the chosen generator.
  Result: Makefile, build.ninja, .vcxproj, etc.

Stage 3: Build (not CMake — your native build tool)
  make, ninja, msbuild, xcodebuild execute the generated build files.
  Result: compiled executables and libraries

The workflow:
  cmake -S . -B build          # configure: source in ., build in build/
  cmake --build build          # build using the generated files
  cmake --build build --target test  # run tests
  cmake --install build        # install to system or prefix

Always use a separate build directory — never build in the source tree.
```

CMake's fundamental concept is the **target**: an executable, library, or custom command. Everything in modern CMake is expressed as target properties. You specify what a target needs, and CMake propagates those requirements automatically.

---

## 3. Progressive Examples

### Level 1: Minimal Project

```cmake
# CMakeLists.txt — the simplest possible C++ project

cmake_minimum_required(VERSION 3.15)  # minimum CMake version required
project(HelloWorld                    # project name
    VERSION 1.0.0                     # optional: sets PROJECT_VERSION
    LANGUAGES CXX)                    # languages used (C, CXX, CUDA, etc.)

# Set C++ standard project-wide
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)   # fail if C++17 not available
set(CMAKE_CXX_EXTENSIONS OFF)         # use -std=c++17 not -std=gnu++17

# Add an executable target
add_executable(hello main.cpp)
```

```bash
# Build commands
mkdir build && cd build
cmake ..                       # configure (generates Makefile by default)
cmake -G Ninja ..              # configure with Ninja generator (faster)
make                           # build (or: cmake --build .)
cmake --build . --parallel 8   # build with 8 parallel jobs

# Modern out-of-tree build (preferred):
cmake -S . -B build            # source=., build dir=build
cmake --build build            # build
./build/hello                  # run
```

### Level 2: Libraries, Dependencies, and Target Properties

```cmake
cmake_minimum_required(VERSION 3.15)
project(MyApp VERSION 2.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Build a library (static by default, or SHARED for .so/.dll)
add_library(mylib STATIC
    src/utils.cpp
    src/database.cpp
    src/networking.cpp
)

# Set include directories on the TARGET (modern CMake — not global)
target_include_directories(mylib
    PUBLIC   include/          # consumers of mylib also see this
    PRIVATE  src/internal/     # only mylib itself sees this
    INTERFACE interface/       # only consumers see this, not mylib itself
)
# PUBLIC: mylib and everything linking against it gets include/
# PRIVATE: only mylib itself gets src/internal/
# INTERFACE: only things linking against mylib get interface/

# Set compiler options
target_compile_options(mylib
    PRIVATE
        -Wall -Wextra          # warnings for library code
        $<$<CONFIG:Debug>:-g -O0>           # debug mode: no opt
        $<$<CONFIG:Release>:-O3 -DNDEBUG>   # release: optimize
)
# $<$<CONFIG:Debug>:...> is a generator expression — evaluated at build time

# Link libraries (mylib needs pthreads)
target_link_libraries(mylib
    PRIVATE Threads::Threads   # CMake's portable threading target
)

# The executable that uses our library
add_executable(myapp
    app/main.cpp
    app/cli_parser.cpp
)

# Link myapp against mylib
# This also propagates mylib's PUBLIC include directories and options to myapp
target_link_libraries(myapp PRIVATE mylib)
```

### Level 3: Finding and Using External Libraries

```cmake
# Modern CMake: use find_package() to locate system libraries
cmake_minimum_required(VERSION 3.15)
project(NetworkApp LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)

# Find system libraries — CMake includes "Find Modules" for common libraries
find_package(OpenSSL REQUIRED)          # fails if not found (REQUIRED)
find_package(Threads REQUIRED)
find_package(ZLIB)                      # optional (no REQUIRED)

# Config-mode packages (provided by the library itself via *Config.cmake)
find_package(Boost 1.70 REQUIRED COMPONENTS filesystem system)
find_package(fmt REQUIRED)              # fmt library (modern C++ formatting)
find_package(nlohmann_json REQUIRED)    # JSON library

add_executable(server
    src/main.cpp
    src/http_server.cpp
    src/tls_handler.cpp
)

target_link_libraries(server
    PRIVATE
        OpenSSL::SSL                # imported target — includes + libs bundled
        OpenSSL::Crypto
        Threads::Threads
        Boost::filesystem
        Boost::system
        fmt::fmt
        nlohmann_json::nlohmann_json
)

# Conditional: only link ZLIB if found
if(ZLIB_FOUND)
    target_link_libraries(server PRIVATE ZLIB::ZLIB)
    target_compile_definitions(server PRIVATE HAVE_ZLIB=1)
endif()

# FetchContent: download dependencies automatically (CMake 3.11+)
include(FetchContent)

FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG        v1.14.0
)
FetchContent_MakeAvailable(googletest)   # downloads, configures, and makes available
```

### Level 4: Testing, Installation, and Packaging

```cmake
cmake_minimum_required(VERSION 3.15)
project(Calculator LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)

# Enable CTest
enable_testing()
include(CTest)

# Main library
add_library(calc STATIC src/calculator.cpp)
target_include_directories(calc PUBLIC include/)

# Main executable
add_executable(calculator app/main.cpp)
target_link_libraries(calculator PRIVATE calc)

# Tests — using GoogleTest (via FetchContent or system install)
include(FetchContent)
FetchContent_Declare(
    GTest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG v1.14.0
)
FetchContent_MakeAvailable(GTest)

add_executable(calc_tests
    tests/test_basic.cpp
    tests/test_edge_cases.cpp
)
target_link_libraries(calc_tests PRIVATE calc GTest::gtest_main)

# Register tests with CTest
include(GoogleTest)
gtest_discover_tests(calc_tests)   # auto-discovers all TEST() macros

# Install rules
include(GNUInstallDirs)   # defines CMAKE_INSTALL_BINDIR, etc.

install(TARGETS calculator calc
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}     # executables
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}     # shared libs
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}     # static libs
)

install(DIRECTORY include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Package configuration for other CMake projects to use
install(EXPORT CalculatorTargets
    FILE CalculatorTargets.cmake
    NAMESPACE Calculator::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/Calculator
)

include(CMakePackageConfigHelpers)
configure_package_config_file(
    cmake/CalculatorConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/CalculatorConfig.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/Calculator
)
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/CalculatorConfig.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/Calculator
)
```

```bash
# Build and test workflow
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build
cd build && ctest --output-on-failure    # run all tests
ctest -R "test_basic"                    # run tests matching pattern
cmake --build build --target install     # install to CMAKE_INSTALL_PREFIX
```

### Level 5: Variables, Cache, and Build Types

```cmake
# Variables
set(MY_VAR "hello")        # set a variable
message(STATUS "var: ${MY_VAR}")  # print during configure

# Cache variables: persist between cmake runs, appear in ccmake/cmake-gui
set(ENABLE_FEATURE ON CACHE BOOL "Enable the extra feature")
set(LOG_LEVEL "INFO" CACHE STRING "Logging level: DEBUG INFO WARN ERROR")
set(MAX_CONNECTIONS "100" CACHE STRING "Maximum connections")

# Option shorthand for BOOL cache variable
option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
option(BUILD_TESTS "Build unit tests" ON)

# Use cache variable
if(ENABLE_FEATURE)
    target_compile_definitions(myapp PRIVATE ENABLE_FEATURE=1)
endif()

# Build types (set with -DCMAKE_BUILD_TYPE=Release)
# Debug: -g, no optimization
# Release: -O3, NDEBUG, no debug symbols
# RelWithDebInfo: -O2, -g, NDEBUG (production with crash symbols)
# MinSizeRel: -Os, NDEBUG (minimize binary size)

# Set default build type if not specified
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Build type" FORCE)
endif()

# List manipulation
set(SOURCES "main.cpp" "util.cpp" "config.cpp")
list(APPEND SOURCES "extra.cpp")         # add to list
list(REMOVE_ITEM SOURCES "config.cpp")  # remove from list
list(LENGTH SOURCES len)                 # get length
foreach(src IN LISTS SOURCES)
    message(STATUS "Source: ${src}")
endforeach()

# String operations
string(TOUPPER "${MY_VAR}" UPPER_VAR)        # → HELLO
string(REPLACE "hello" "world" NEW_VAR "${MY_VAR}")
string(REGEX MATCH "[0-9]+" num "version123")  # → 123

# File operations
file(GLOB_RECURSE SOURCES "src/*.cpp")    # collect all .cpp files
# WARNING: GLOB doesn't track new files — prefer explicit file lists
# Or use CONFIGURE_DEPENDS (CMake 3.12+):
file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS "src/*.cpp")
```

### Level 6: Multi-Directory Projects and Best Practices

```
Project structure (recommended):
├── CMakeLists.txt          ← root: project setup, subdirs, options
├── cmake/
│   └── FindCustomLib.cmake ← custom Find modules
│   └── ProjectConfig.cmake.in
├── src/
│   ├── CMakeLists.txt      ← library target
│   └── *.cpp
├── include/
│   └── mylib/
│       └── *.hpp
├── app/
│   ├── CMakeLists.txt      ← executable target
│   └── main.cpp
└── tests/
    ├── CMakeLists.txt      ← test targets
    └── *.cpp
```

```cmake
# Root CMakeLists.txt
cmake_minimum_required(VERSION 3.15)
project(MyProject VERSION 1.2.3 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Put all binaries in one place
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# Add custom cmake module directory
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

# Project-wide compiler warnings
add_library(project_warnings INTERFACE)  # INTERFACE: no compiled files
target_compile_options(project_warnings INTERFACE
    $<$<CXX_COMPILER_ID:GNU,Clang>:-Wall -Wextra -Wpedantic -Werror>
    $<$<CXX_COMPILER_ID:MSVC>:/W4 /WX>
)

# Address sanitizer (for Debug builds)
option(ENABLE_ASAN "Enable AddressSanitizer" OFF)
if(ENABLE_ASAN)
    add_compile_options(-fsanitize=address,undefined)
    add_link_options(-fsanitize=address,undefined)
endif()

# Add subdirectories
add_subdirectory(src)    # defines target: mylib
add_subdirectory(app)    # defines target: myapp
if(BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()

# src/CMakeLists.txt
add_library(mylib STATIC calculator.cpp networking.cpp)
target_include_directories(mylib PUBLIC "${PROJECT_SOURCE_DIR}/include")
target_link_libraries(mylib PRIVATE project_warnings)

# app/CMakeLists.txt
add_executable(myapp main.cpp)
target_link_libraries(myapp PRIVATE mylib project_warnings)
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Using global `include_directories()` and `link_libraries()` instead of target-specific versions**

```cmake
# WRONG: affects all targets in scope — causes unintended side effects
include_directories(include/)
link_libraries(OpenSSL::SSL)

# CORRECT: specify per-target with appropriate visibility
target_include_directories(mytarget PRIVATE include/)
target_link_libraries(mytarget PRIVATE OpenSSL::SSL)
```

**Mistake 2: Globbing source files without CONFIGURE_DEPENDS**

```cmake
# WRONG: CMake won't re-run configure when new files are added!
file(GLOB SOURCES "src/*.cpp")

# BETTER: use CONFIGURE_DEPENDS (slower but detects new files)
file(GLOB SOURCES CONFIGURE_DEPENDS "src/*.cpp")

# BEST: list files explicitly
add_library(mylib
    src/utils.cpp
    src/database.cpp
    # New files: add them manually here
)
```

**Mistake 3: Building in the source directory**

```bash
# WRONG: pollutes source tree with build artifacts
cd /project
cmake .
make    # creates Makefile, CMakeCache.txt, CMakeFiles/ in source dir

# CORRECT: always use a separate build directory
cmake -S . -B build
cmake --build build
```

**Mistake 4: Not specifying `cmake_minimum_required()`**

Without `cmake_minimum_required()`, CMake uses compatibility policies from an ancient version, which changes the behavior of many commands. Always specify the minimum version you actually need.

---

## 5. The "Why Does This Work" Layer

### Why Modern CMake Uses Targets Instead of Variables

Old CMake (< 3.0 style) used global variables: `include_directories()` added to a global list that all subsequent targets inherited. This caused include path pollution across targets and made it impossible to have different targets with different settings in the same directory.

Modern CMake (3.0+ "target-based CMake") treats each target as a self-contained unit with its own properties. When you link target A to target B, CMake automatically propagates B's PUBLIC properties to A. This is how a library can declare "anyone linking me also needs to know about my include directory" without you manually passing that information to every consumer.

### How Generator Expressions Enable Multi-Configuration Builds

A generator expression like `$<$<CONFIG:Release>:-O3>` is NOT evaluated during configuration. It's a placeholder that gets evaluated when the build files are generated, once per build configuration. This allows a single CMakeLists.txt to produce both Debug and Release variants without if-else logic, which is essential for multi-configuration generators like Visual Studio that build Debug and Release simultaneously.

---

## 6. Quick Reference

### Core Commands

```cmake
cmake_minimum_required(VERSION 3.15)   # required first line
project(Name LANGUAGES CXX)            # project name and languages
add_executable(target files...)         # create executable
add_library(target STATIC/SHARED files...) # create library
target_include_directories(t PUBLIC ...) # include paths
target_link_libraries(t PRIVATE ...)    # link dependencies
target_compile_options(t PRIVATE ...)   # compiler flags
target_compile_definitions(t PRIVATE NAME=VALUE) # preprocessor defs
```

### Workflow

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release  # configure
cmake --build build --parallel $(nproc)          # build
cmake --build build --target test                # test
cmake --install build --prefix /usr/local        # install

# Debug build
cmake -S . -B debug -DCMAKE_BUILD_TYPE=Debug -DENABLE_ASAN=ON
cmake --build debug
```

### Variables Reference

| Variable | Meaning |
|----------|---------|
| `CMAKE_SOURCE_DIR` | Root CMakeLists.txt directory |
| `CMAKE_BINARY_DIR` | Root build directory |
| `PROJECT_SOURCE_DIR` | Current project's CMakeLists.txt |
| `CMAKE_CURRENT_SOURCE_DIR` | Current CMakeLists.txt directory |
| `CMAKE_BUILD_TYPE` | Debug/Release/RelWithDebInfo/MinSizeRel |
| `CMAKE_INSTALL_PREFIX` | Installation root (default: /usr/local) |
| `BUILD_SHARED_LIBS` | Global default for add_library |
