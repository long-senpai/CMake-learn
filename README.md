# Step 1: A Basic Starting Point

## A simple Project

```cmake
# CMakeLists.txt
# ----------------------------------
cmake_minimum_required(VERSION 3.10)

# set the project name
project(Tutorial)

# add the executable
add_executable(Tutorial tutorial.cxx)
```

- Upper, lower, and mixed case commands are supported by CMake.

## Adding a Version Number and Configured Header File

```cmake
# CMakeLists.txt
# -----------------------------------------------------------------
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

configure_file(TutorialConfig.h.in TutorialConfig.h)
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
```

```cpp
// TutorialConfig.h.in
// ----------------------------------------------------
// the configured options and settings for Tutorial
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

```cpp
// tutorial.cxx
// ---------------------------------------------------------------------
if (argc < 2) {
    // report version
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
}
```

## Specify the C++ Standard

```cmake
# CMakeLists.txt
# ------------------------------------
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

## Build and Test

```bash
>> mkdir build
>> cd build
>> cmake ..
>> cmake --build .
```

# Step2: Adding a Library

## Link

Assume in the subdirectory `MathFunctions`, it already contains a header file, `MathFunctions.h`, and a source file `mysqrt.cxx`.
```
Directory tree
------------------------

Step2
├── build
├── CMakeLists.txt
├── MathFunctions
│   ├── MathFunctions.h
│   └── mysqrt.cxx
├── TutorialConfig.h.in
└── tutorial.cxx
```

- Add library by one line

```cmake
# MathFunctions/CMakeLists.txt
# -----------------------------------
add_library(MathFunctions mysqrt.cxx)
```

- To make use of the new library we will add an `add_subdirectory()` call in the top-level `CMakeLists.txt` file so that the library will get built.

- Add the new library to the executable, and add `MathFunctions` as an include directory so that the `mysqrt.h` header file can be found

```cmake
# CMakeLists.txt
# --------------------------------------------------------------
# add the MathFunctions library
add_subdirectory(MathFunctions)

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC MathFunctions)

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                          "${PROJECT_BINARY_DIR}"
                          "${PROJECT_SOURCE_DIR}/MathFunctions"
                          )
```

- `add_subdirectory(source_dir [binary_dir])`: add a directory to the build `source_dir`: specifies directory in `CMakeLists.txt` and code file located. `binary_dir`: for output file CMakeLists.txt bên trong `src_dir` sẽ được xử lý bằng `CMake` trước.

- `target_include-directories(<target> [system] [AFTER|BEFORE] [<INTERFACE|PUBLIC|PRIVATE> [item ...] ...])`: specifies include directories to use when compiling a given target. `<target>` been created such as `add_executable()` and `add_library()`.

- `target_link_libraries(<target> [<INTERFACE|PUBLIC|PRIVATE> <item> ...])`: link libraries for targets and it dependents.

## Optional

```cmake
# CMakeLists.txt
# -------------------------------------------------------------------
if(USE_MYMATH)
    add_subdirectory(MathFunctions)
    list(APPEND EXTRA_LIBS MathFunctions)
    list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif()

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           ${EXTRA_INCLUDES})
```

```cpp
// tutorial.cxx
// ---------------------------------------------
#ifdef USE_MYMATH
#  include "MathFunctions.h"
#endif

#ifdef USE_MYMATH
  const double outputValue = mysqrt(inputValue);
#else
  const double outputValue = sqrt(inputValue);
#endif
```

```cpp
// TutorialConfig.h.in
// --------------------
#cmakedefine USE_MYMATH
```

```bash
cmake .. -DUSE_MYMATH=OFF
```

# Step 3: Adding Usage Requirements for a Library

- [target_compile_definitions()](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions)
- [target_compile_options()](https://cmake.org/cmake/help/latest/command/target_compile_options.html#command:target_compile_options)
- [target_include_directories()](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories)
- [target_link_libraries()](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries)

# Step 4: Installing and Testing

## Install Rules

```cmake
# MathFunctions/CMakeLists.txt
# ------------------------------------------------
install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```

```cmake
# CMakeLists.txt
# -------------------------------------------------------------------------
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h" DESTINATION include)
```

```bash
cmake --install .
```

- Navigate to the install directory.

```bash
cmake --install . --prefix "/home/myuser/installdir"
```

## Testing Support

```cmake
# CMakeLists.txt
# -------------------------------------------------------------------------
enable_testing()

# does the application run
add_test(NAME Runs COMMAND Tutorial 25)

# does the usage message work?
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number")

# define a function to simplify adding tests
function(do_test target arg result)
    add_test(NAME Comp${arg} COMMAND ${target} ${arg})
    set_tests_properties(Comp${arg} PROPERTIES PASS_REGULAR_EXPRESSION ${result}
)
endfunction(do_test)

# do a bunch of result based tests
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is [-nan|nan|0]")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
```

[link](https://cmake.org/cmake/help/latest/guide/tutorial/Installing%20and%20Testing.html#testing-support)

