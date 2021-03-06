cmake_minimum_required(VERSION 3.4.3)

if (POLICY CMP0048)
  cmake_policy(SET CMP0048 NEW)
  set(PROJ_VERSION VERSION 1.5.0)
endif()

project(OpenCL-ROCm)

# Add path for custom modules
set(CMAKE_MODULE_PATH
  ${CMAKE_MODULE_PATH}
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake"
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules"
  )

set(LLVM_INCLUDE_TESTS CACHE BOOL OFF)
set(LLVM_TARGETS_TO_BUILD "AMDGPU" CACHE STRING "")
set(LLVM_ENABLE_PROJECTS CACHE STRING "clang")
set(LLVM_BUILD_GLOBAL_ISEL CACHE BOOL OFF)
set(CLANG_ENABLE_ARCMT CACHE BOOL OFF)
set(CLANG_ENABLE_STATIC_ANALYZER CACHE BOOL OFF)

add_subdirectory(compiler/llvm)

find_package(LLVM REQUIRED CONFIG PATHS ${CMAKE_BINARY_DIR}/compiler/llvm NO_DEFAULT_PATH)

list(APPEND CMAKE_MODULE_PATH "${LLVM_CMAKE_DIR}")
include(AddLLVM)

add_definitions(${LLVM_DEFINITIONS})
# TODO: add find_package(Clang) and use LLVM/Clang variables got from their config
include_directories(${CMAKE_SOURCE_DIR}/compiler/llvm/tools/clang/include)
include_directories(${CMAKE_BINARY_DIR}/compiler/llvm/tools/clang/include)

# TODO: move AMDGPU.h header to include folder
include_directories(${CMAKE_SOURCE_DIR}/compiler/llvm/lib/Target/AMDGPU)
include_directories(${CMAKE_BINARY_DIR}/compiler/llvm/lib/Target/AMDGPU)

add_subdirectory(compiler/driver)

set(BUILD_HC_LIB CACHE BOOL OFF)
set(ROCM_DEVICELIB_INCLUDE_TESTS CACHE BOOL OFF)
add_subdirectory(library/amdgcn)

add_subdirectory(compiler/lib/loaders/elf/utils/libelf)

add_subdirectory(tools/bc2h)

find_package(ROCR REQUIRED)
add_subdirectory(runtime)
add_subdirectory(api/opencl/amdocl)
