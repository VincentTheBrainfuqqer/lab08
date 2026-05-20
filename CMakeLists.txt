cmake_minimum_required(VERSION 3.4)

include("cmake/HunterGate.cmake")

HunterGate(
    URL "https://github.com/cpp-pm/hunter/archive/v0.23.251.tar.gz"
    SHA1 "5659b15dc0884d4b03dbd95710e6a1fa0fc3258d"
    LOCAL
)

project(print)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(PRINT_VERSION_MAJOR 0)
set(PRINT_VERSION_MINOR 1)
set(PRINT_VERSION_PATCH 0)
set(PRINT_VERSION_TWEAK 0)

set(PRINT_VERSION
    ${PRINT_VERSION_MAJOR}.${PRINT_VERSION_MINOR}.${PRINT_VERSION_PATCH}.${PRINT_VERSION_TWEAK}
)

set(PRINT_VERSION_STRING "v${PRINT_VERSION}")

hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)

option(BUILD_TESTS "Build tests" OFF)

add_library(print STATIC sources/print.cpp)
target_include_directories(print PUBLIC include)

add_executable(example1 examples/example1.cpp)
target_link_libraries(example1 print)

add_executable(example2 examples/example2.cpp)
target_link_libraries(example2 print)

add_executable(solver examples/example1.cpp)
target_link_libraries(solver print)

if(BUILD_TESTS)
    enable_testing()

    file(GLOB TEST_SOURCES tests/*.cpp)

    add_executable(check ${TEST_SOURCES})
    target_link_libraries(check print GTest::main)

    add_test(NAME check COMMAND check)
endif()

install(TARGETS print
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

install(TARGETS solver
    RUNTIME DESTINATION bin
)

install(FILES include/print.hpp
    DESTINATION include
)

add_executable(demo ${CMAKE_CURRENT_SOURCE_DIR}/demo/main.cpp)
target_link_libraries(demo print)

install(TARGETS demo
    RUNTIME DESTINATION bin
)

include(CPackConfig.cmake)
