cmake_minimum_required(VERSION 3.16)

project(cpplox
  VERSION 0.0.1
  DESCRIPTION "A C++ implementation of a compiler for the Lox language"
  LANGUAGES CXX)

# set the C++ standard
set(CMAKE_CXX_STANDARD 20)

# generate the compile_commands.json
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# enable all warnings and treat them as errors
add_compile_options(-Wall -Wextra -pedantic)
add_compile_options(-Werror)
# enable clang-tidy checks
set(CMAKE_CXX_CLANG_TIDY clang-tidy --use-color)

set(allowedBuildTypes Debug Release ASAN MSAN UBSAN TSAN Profile HeapProfile)
set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "${allowedBuildTypes}")
if(CMAKE_BUILD_TYPE AND NOT CMAKE_BUILD_TYPE IN_LIST allowedBuildTypes)
  message(FATAL_ERROR "Invalid build type: ${CMAKE_BUILD_TYPE}")
endif()

#add_compile_options(-ftemplate-backtrace-limit=0)
set(CMAKE_CXX_FLAGS_DEBUG "-g -O0" CACHE STRING "Debug build" FORCE)
set(CMAKE_CXX_FLAGS_RELEASE "-DNDEBUG -O3" CACHE STRING "Release build" FORCE)
set(CMAKE_CXX_FLAGS_ASAN "-fsanitize=address -fno-omit-frame-pointer -fno-optimize-sibling-calls -g -O1" CACHE STRING "Address Sanitizer build" FORCE)
set(CMAKE_CXX_FLAGS_MSAN "-fsanitize=memory -fsanitize-memory-track-origins -fno-omit-frame-pointer -fno-optimize-sibling-calls -g -O1" CACHE STRING "Memory Sanitizer build" FORCE)
set(CMAKE_CXX_FLAGS_UBSAN "-fsanitize=undefined -fno-omit-frame-pointer -fno-optimize-sibling-calls -g -O0" CACHE STRING "Undefined Behaviour Sanitizer" FORCE)
set(CMAKE_CXX_FLAGS_TSAN "-fsanitize=thread -g -O0" CACHE STRING "Thread Sanitizer" FORCE)
set(CMAKE_CXX_FLAGS_PROFILE "-pg -g" CACHE STRING "gprof profiler" FORCE)
set(CMAKE_CXX_FLAGS_HEAPPROFILE "-g -O0" CACHE STRING "Heap Profiler" FORCE)

if("${CMAKE_BUILD_TYPE}" STREQUAL "MSAN" AND "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
  message(FATAL_ERROR "Memory sanitizer is not currently supported by gcc. Try clang instead.")
endif()

set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

add_subdirectory(src)
