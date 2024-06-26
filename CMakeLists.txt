cmake_minimum_required(VERSION 3.16)

project(
  cpplox
  VERSION 0.0.1
  DESCRIPTION "A C++ implementation of a compiler for the Lox language"
  LANGUAGES CXX)

# set the C++ standard
set(CMAKE_CXX_STANDARD 20)

# generate the compile_commands.json
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

link_directories(${RHOME}/lib ${RHOME}/lib64)

set(CMAKE_INSTALL_RPATH_USE_LINK_PATH ON)
set(CMAKE_BUILD_RPATH "$ENV{RHOME}/lib;$ENV{RHOME}/lib64")
set(CMAKE_INSTALL_RPATH "$ENV{RHOME}/lib;$ENV{RHOME}/lib64")

function(target_add_warnings target)
  # enable all warnings
  target_compile_options(${target} PRIVATE -Wall -Wextra -Wshadow -Wnon-virtual-dtor -pedantic)
  # https://github.com/cpp-best-practices/cppbestpractices/blob/master/02-Use_the_Tools_Available.md#gcc--clang
  target_compile_options(
    ${target}
    PRIVATE -Wold-style-cast
            -Wcast-align
            -Wunused
            -Woverloaded-virtual
            -Wconversion
            -Wsign-conversion
            -Wmisleading-indentation
            -Wnull-dereference
            -Wdouble-promotion
            -Wformat=2
            -Wimplicit-fallthrough)
  if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    # using Clang
  elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    target_compile_options(${target} PRIVATE -Wduplicated-branches -Wduplicated-cond -Wlogical-op -Wuseless-cast)
  endif()
  # treat all warnings as errors
  target_compile_options(${target} PRIVATE -Werror)
endfunction()

set(allowedBuildTypes
    Debug
    Release
    ASAN
    MSAN
    UBSAN
    TSAN
    Profile
    HeapProfile)
set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "${allowedBuildTypes}")
if(CMAKE_BUILD_TYPE AND NOT CMAKE_BUILD_TYPE IN_LIST allowedBuildTypes)
  message(FATAL_ERROR "Invalid build type: ${CMAKE_BUILD_TYPE}")
endif()

# add_compile_options(-ftemplate-backtrace-limit=0)
set(CMAKE_CXX_FLAGS_DEBUG
    "-g -O0"
    CACHE STRING "Debug build" FORCE)
set(CMAKE_CXX_FLAGS_RELEASE
    "-DNDEBUG -O3"
    CACHE STRING "Release build" FORCE)
set(CMAKE_CXX_FLAGS_ASAN
    "-fsanitize=address -fno-omit-frame-pointer -fno-optimize-sibling-calls -g -O1"
    CACHE STRING "Address Sanitizer build" FORCE)
set(CMAKE_CXX_FLAGS_MSAN
    "-fsanitize=memory -fsanitize-memory-track-origins -fno-omit-frame-pointer -fno-optimize-sibling-calls -g -O1 -stdlib=libc++ -I/home/night/.local/llvm-msan/include -I/home/night/.local/llvm-msan/include/c++/v1"
    CACHE STRING "Memory Sanitizer build" FORCE)
set(CMAKE_CXX_FLAGS_UBSAN
    "-fsanitize=undefined -fno-omit-frame-pointer -fno-optimize-sibling-calls -g -O0"
    CACHE STRING "Undefined Behaviour Sanitizer" FORCE)
set(CMAKE_CXX_FLAGS_TSAN
    "-fsanitize=thread -g -O0"
    CACHE STRING "Thread Sanitizer" FORCE)
set(CMAKE_CXX_FLAGS_PROFILE
    "-pg -g"
    CACHE STRING "gprof profiler" FORCE)
set(CMAKE_CXX_FLAGS_HEAPPROFILE
    "-g -O0"
    CACHE STRING "Heap Profiler" FORCE)

if("${CMAKE_BUILD_TYPE}" STREQUAL "MSAN" AND "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
  message(FATAL_ERROR "Memory sanitizer is not currently supported by gcc. Try clang instead.")
endif()
if("${CMAKE_BUILD_TYPE}" STREQUAL "MSAN")
  add_link_options(
    -fsanitize=memory
    -fsanitize-memory-track-origins
    -fno-omit-frame-pointer
    -fno-optimize-sibling-calls
    -g
    -stdlib=libc++
    -L/home/night/.local/llvm-msan/lib
    -Wl,-rpath=/home/night/.local/llvm-msan/lib
    -lc++abi)
endif()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

include(FetchContent)
FetchContent_Declare(
  fmt
  GIT_REPOSITORY https://github.com/fmtlib/fmt.git
  GIT_TAG 10.2.1
  OVERRIDE_FIND_PACKAGE)
FetchContent_Declare(
  Catch2
  GIT_REPOSITORY https://github.com/catchorg/Catch2.git
  GIT_TAG v3.6.0
  OVERRIDE_FIND_PACKAGE)
list(APPEND CMAKE_MODULE_PATH ${catch2_SOURCE_DIR}/extras)
find_package(fmt REQUIRED)
find_package(Catch2 3 REQUIRED)

add_subdirectory(src)
add_subdirectory(tests)
