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

# Step 5: Adding System Introspection

Let us consider adding some code to our project that depends on features the target platform may not have. For this example, we will add some code that depends on whether or not the target platform has the `log` and `exp` functions. Of course almost every platform has these functions but for this tutorial assume that they are not common.

If the platform has `log` and `exp` then we will use them to compute the square root in the `mysqrt` function. We first test for the availability of these functions using the `CheckSymbolExists` module in `MathFunctions/CMakeLists.txt`. On some platforms, we will need to link to the `m` library. If `log` and `exp` are not initially found, require the `m` library and try again.

```cmake
# MathFunctions/CMakeLists.txt
# -----------------------------------------------------
include(CheckSymbolExists)
check_symbol_exists(log "math.h" HAVE_LOG)
check_symbol_exists(exp "math.h" HAVE_EXP)
if(NOT (HAVE_LOG AND HAVE_EXP))
    unset(HAVE_LOG CACHE)
    unset(HAVE_EXP CACHE)
    set(CMAKE_REQUIRED_LIBRARIES "m")
    check_symbol_exists(log "math.h" HAVE_LOG)
    check_symbol_exists(exp "math.h" HAVE_EXP)
    if(HAVE_LOG AND HAVE_EXP)
        target_link_libraries(MathFunctions PRIVATE m)
    endif()
endif()
```

```cpp
// MathFunctions/mysqrt.cxx
// -------------------------------------------------------------
#include <cmath>

#if defined(HAVE_LOG) && defined(HAVE_EXP)
    double result = exp(log(x) * 0.5);
    std::cout << "Computing sqrt of " << x << " to be " << result
              << " using log and exp" << std::endl;
#else
    double result = x;
```

# Step 6: Adding a Custom Command and Generated File

Suppose, for the purpose of this tutorial, we decide that we never want to use the platform `log` and `exp` functions and instead would like to generate a table of precomputed values to use in the `mysqrt` function. In this section, we will create the table as part of the build process, and then compile that table into our application.

First, let's remove the check for the `log` and `exp` functions in `MathFunctions/CMakeLists.txt`. Then remove the check for `HAVE_LOG` and `HAVE_EXP` from `mysqrt.cxx`. At the same time, we can remove `#include <cmath>`.

In the `MathFunctions` subdirectory, a new source file named `MakeTable.cxx` has been provided to generate the table.

After reviewing the file, we can see that the table is produced as valid C++ code and that the output filename is passed in as an argument.

The next step is to add the appropriate commands to the `MathFunctions/CMakeLists.txt` file to build the MakeTable executable and then run it as part of the build process. A few commands are needed to accomplish this.

```cmake
# MathFunctions/CMakeLists.txt
# -----------------------------------------------
add_executable(MakeTable MakeTable.cxx)
add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
    COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
    DEPENDS MakeTable
)

add_library(MathFunctions
            mysqrt.cxx
            ${CMAKE_CURRENT_BINARY_DIR}/Table.h
            )
```

# Step 7: Packaging an Installer

Next suppose that we want to distribute our project to other people so that they can use it. We want to provide both binary and source distributions on a variety of platforms. This is a little different from the install we did previously in Installing and Testing, where we were installing the binaries that we had built from the source code. In this example we will be building installation packages that support binary installations and package management features. To accomplish this we will use CPack to create platform specific installers. Specifically we need to add a few lines to the bottom of our top-level CMakeLists.txt file.

```cmake
# CMakeLists.txt
# ------------------------------------------------------------------------
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
include(CPack)
```

That is all there is to it. We start by including InstallRequiredSystemLibraries. This module will include any runtime libraries that are needed by the project for the current platform. Next we set some CPack variables to where we have stored the license and version information for this project. The version information was set earlier in this tutorial and the license.txt has been included in the top-level source directory for this step.

Finally we include the CPack module which will use these variables and some other properties of the current system to setup an installer.

The next step is to build the project in the usual manner and then run the cpack executable. To build a binary distribution, from the binary directory run:

```bash
cpack
```

To specify the generator, use the -G option. For multi-config builds, use -C to specify the configuration. For example:

```bash
cpack -G ZIP -C Debug
```

To create a source distribution you would type:

```bash
cpack --config CPackSourceConfig.cmake
```

Alternatively, run make package or right click the Package target and Build Project from an IDE.
Run the installer found in the binary directory. Then run the installed executable and verify that it works.

# Step 8: Adding Support for a Testing Dashboard

[link](https://cmake.org/cmake/help/latest/guide/tutorial/Adding%20Support%20for%20a%20Testing%20Dashboard.html)

# Step 9: Selecting Static or Shared Libraries

In this section we will show how the `BUILD_SHARED_LIBS` variable can be used to control the default behavior of `add_library()`, and allow control over how libraries without an explicit type (`STATIC`, `SHARED`, `MODULE` or `OBJECT`) are built.

To accomplish this we need to add `BUILD_SHARED_LIBS` to the top-level CMakeLists.txt. We use the `option()` command as it allows users to optionally select if the value should be `ON` or `OFF`.

```cmake
# CMakeLists.txt
# ----------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# control where the static and shared libraries are built so that on windows
# we don't need to tinker with the path to run the executable
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")

option(BUILD_SHARED_LIBS "Build using shared libraries" ON)

# configure a header file to pass the version number only
configure_file(TutorialConfig.h.in TutorialConfig.h)

# add the MathFunctions library
add_subdirectory(MathFunctions)

# add the executable
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions)
```

```cmake
# MathFunctions/CMakeLists.txt
# ----------------------------------------------------------------------------
# add the library that runs
add_library(MathFunctions MathFunctions.cxx)

# state that anybody linking to us needs to include the current source dir
# to find MathFunctions.h, while we don't.
target_include_directories(MathFunctions
                           INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
                           )

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if(USE_MYMATH)
    target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")

    # first we add the executable that generates the table
    add_executable(MakeTable MakeTable.cxx)

    # add the command to generate the source code
    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
        COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
        DEPENDS MakeTable
    )

    # library that just does sqrt
    add_library(SqrtLibrary STATIC
                mysqrt.cxx
                ${CMAKE_CURRENT_BINARY_DIR}/Table.h
                )

    # state that we depend on our binary dir to find Table.h
    target_include_directories(SqrtLibrary PRIVATE
                                ${CMAKE_CURRENT_BINARY_DIR}
                                )

    target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()

# define the symbol stating we are using the declspec(dllexport) when
# building on windows
target_compile_definitions(MathFunctions PRIVATE "EXPORTING_MYMATH")

# install rules
set(installable_libs MathFunctions)
if(TARGET SqrtLibrary)
    list(APPEND installable_libs SqrtLibrary)
endif()

install(TARGETS ${installable_libs} DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```

```cpp
// MathFunctions/mysqrt.cxx
// ----------------------------------------------------------------------------------------
#include <iostream>
#include "MathFunctions.h"

// include the generated table
#include "Table.h"

namespace mathfunctions
{
    namespace detail
    {
        // a hack square root calculation using simple operations
        double mysqrt(double x)
        {
            if (x <= 0)
            {
                return 0;
            }

            // use the table to help find an initial value
            double result = x;
            if (x >= 1 && x < 10)
            {
                std::cout << "Use the table to help find an initial value " << std::endl;
                result = sqrtTable[static_cast<int>(x)];
            }

            // do ten iterations
            for (int i = 0; i < 10; ++i)
            {
                if (result <= 0)
                {
                    result = 0.1;
                }
                double delta = x - (result * result);
                result = result + 0.5 * delta / result;
                std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
            }
            return result;
        }
    }
}
```

```cpp
// MathFunctions/MathFunctions.h
// ----------------------------------------

#if defined(_WIN32)
#  if defined(EXPORTING_MYMATH)
#    define DECLSPEC __declspec(dllexport)
#  else
#    define DECLSPEC __declspec(dllimport)
#  endif
#else // non windows
#  define DECLSPEC
#endif

namespace mathfunctions
{
    double DECLSPEC sqrt(double x);
}
```

```cmake
# MathFunctions/CMakeLists.txt
# -------------------------------------------------------------------
# state that SqrtLibrary need PIC when the default is shared libraries
set_target_properties(SqrtLibrary PROPERTIES
                    POSITION_INDEPENDENT_CODE ${BUILD_SHARED_LIBS}
                    )

target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
```

# Step 10: Adding Generator Expressions

[link](https://cmake.org/cmake/help/latest/guide/tutorial/Adding%20Generator%20Expressions.html)

# Step 11: Adding Export Configuration

[link](https://cmake.org/cmake/help/latest/guide/tutorial/Adding%20Export%20Configuration.html)

# Step 12: Packaging Debug and Release

```bash
cd debug
cmake -DCMAKE_BUILD_TYPE=Debug ..
cmake --build .
cd ../release
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
```

[link](https://cmake.org/cmake/help/latest/guide/tutorial/Packaging%20Debug%20and%20Release.html)