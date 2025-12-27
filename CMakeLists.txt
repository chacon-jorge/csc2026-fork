# CSC Latin America 2026 - HEP Computing
# CMake template for exercises
cmake_minimum_required(VERSION 3.16)
project(CSC2026_Exercises VERSION 1.0.0 LANGUAGES CXX)

# C++20 standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Export compile_commands.json for IDE support
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Build type (default to Release with Debug info)
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Build type" FORCE)
endif()

# Compiler warnings
add_compile_options(
    -Wall
    -Wextra
    -Wpedantic
    -Wshadow
    -Wnon-virtual-dtor
    -Wold-style-cast
    -Wcast-align
    -Woverloaded-virtual
)

# ============================================================================
# Options
# ============================================================================
option(ENABLE_SANITIZERS "Enable ASan and UBSan" OFF)
option(ENABLE_TSAN "Enable ThreadSanitizer (mutually exclusive with ASan)" OFF)
option(ENABLE_OPENMP "Enable OpenMP support" ON)
option(BUILD_TESTS "Build unit tests" ON)
option(BUILD_BENCHMARKS "Build benchmarks" ON)

# ============================================================================
# Sanitizers
# ============================================================================
if(ENABLE_SANITIZERS AND ENABLE_TSAN)
    message(FATAL_ERROR "Cannot enable both ASan and TSan simultaneously")
endif()

if(ENABLE_SANITIZERS)
    message(STATUS "Enabling AddressSanitizer and UndefinedBehaviorSanitizer")
    add_compile_options(-fsanitize=address,undefined -fno-omit-frame-pointer -g)
    add_link_options(-fsanitize=address,undefined)
endif()

if(ENABLE_TSAN)
    message(STATUS "Enabling ThreadSanitizer")
    add_compile_options(-fsanitize=thread -fno-omit-frame-pointer -g)
    add_link_options(-fsanitize=thread)
endif()

# ============================================================================
# Find Dependencies
# ============================================================================

# ROOT
find_package(ROOT REQUIRED COMPONENTS Core RIO Tree Hist MathCore Physics)
include(${ROOT_USE_FILE})
message(STATUS "Found ROOT: ${ROOT_VERSION} at ${ROOT_DIR}")

# OpenMP
if(ENABLE_OPENMP)
    find_package(OpenMP)
    if(OpenMP_CXX_FOUND)
        message(STATUS "Found OpenMP: ${OpenMP_CXX_VERSION}")
    else()
        message(WARNING "OpenMP not found, parallel features disabled")
    endif()
endif()

# TBB
find_package(TBB QUIET)
if(TBB_FOUND)
    message(STATUS "Found TBB: ${TBB_VERSION}")
else()
    message(STATUS "TBB not found, some parallel features may be unavailable")
endif()

# Catch2 (for testing)
if(BUILD_TESTS)
    find_package(Catch2 3 QUIET)
    if(Catch2_FOUND)
        message(STATUS "Found Catch2: ${Catch2_VERSION}")
    else()
        message(STATUS "Catch2 not found, fetching from GitHub...")
        include(FetchContent)
        FetchContent_Declare(
            Catch2
            GIT_REPOSITORY https://github.com/catchorg/Catch2.git
            GIT_TAG v3.5.2
        )
        FetchContent_MakeAvailable(Catch2)
    endif()
endif()

# Google Benchmark
if(BUILD_BENCHMARKS)
    find_package(benchmark QUIET)
    if(benchmark_FOUND)
        message(STATUS "Found Google Benchmark")
    else()
        message(STATUS "Google Benchmark not found, fetching from GitHub...")
        include(FetchContent)
        FetchContent_Declare(
            benchmark
            GIT_REPOSITORY https://github.com/google/benchmark.git
            GIT_TAG v1.8.3
        )
        set(BENCHMARK_ENABLE_TESTING OFF CACHE BOOL "" FORCE)
        FetchContent_MakeAvailable(benchmark)
    endif()
endif()

# ============================================================================
# Library: Core analysis library
# ============================================================================
add_library(analysis SHARED
    src/Particle.cpp
    src/EventProcessor.cpp
    src/TrackReconstructor.cpp
)

target_include_directories(analysis PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${ROOT_INCLUDE_DIRS}
)

target_link_libraries(analysis PUBLIC
    ROOT::Core
    ROOT::RIO
    ROOT::Tree
    ROOT::Hist
    ROOT::MathCore
    ROOT::Physics
)

if(OpenMP_CXX_FOUND)
    target_link_libraries(analysis PUBLIC OpenMP::OpenMP_CXX)
endif()

if(TBB_FOUND)
    target_link_libraries(analysis PUBLIC TBB::tbb)
endif()

# ============================================================================
# Executables
# ============================================================================

# Main analysis executable
add_executable(analyze src/main.cpp)
target_link_libraries(analyze PRIVATE analysis)

# Event processor example
add_executable(event_processor src/event_processor_main.cpp)
target_link_libraries(event_processor PRIVATE analysis)

# ============================================================================
# Tests
# ============================================================================
if(BUILD_TESTS)
    enable_testing()
    
    add_executable(tests
        tests/test_particle.cpp
        tests/test_event_processor.cpp
        tests/test_track_reconstructor.cpp
    )
    
    target_link_libraries(tests PRIVATE
        analysis
        Catch2::Catch2WithMain
    )
    
    # Register tests with CTest
    include(Catch)
    catch_discover_tests(tests)
endif()

# ============================================================================
# Benchmarks
# ============================================================================
if(BUILD_BENCHMARKS)
    add_executable(benchmarks
        benchmarks/bench_particle.cpp
        benchmarks/bench_event_processor.cpp
    )
    
    target_link_libraries(benchmarks PRIVATE
        analysis
        benchmark::benchmark
        benchmark::benchmark_main
    )
endif()

# ============================================================================
# Installation
# ============================================================================
install(TARGETS analysis analyze
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)

install(DIRECTORY include/
    DESTINATION include
)

# ============================================================================
# Print Configuration Summary
# ============================================================================
message(STATUS "")
message(STATUS "=== CSC 2026 Build Configuration ===")
message(STATUS "Build type:        ${CMAKE_BUILD_TYPE}")
message(STATUS "C++ Standard:      ${CMAKE_CXX_STANDARD}")
message(STATUS "Compiler:          ${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "ROOT version:      ${ROOT_VERSION}")
message(STATUS "OpenMP:            ${OpenMP_CXX_FOUND}")
message(STATUS "TBB:               ${TBB_FOUND}")
message(STATUS "Sanitizers:        ${ENABLE_SANITIZERS}")
message(STATUS "ThreadSanitizer:   ${ENABLE_TSAN}")
message(STATUS "Build tests:       ${BUILD_TESTS}")
message(STATUS "Build benchmarks:  ${BUILD_BENCHMARKS}")
message(STATUS "=====================================")
message(STATUS "")
