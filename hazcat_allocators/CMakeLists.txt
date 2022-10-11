cmake_minimum_required(VERSION 3.8)
set(CMAKE_CUDA_COMPILER /usr/local/cuda/bin/nvcc)
project(hazcat_allocators CUDA CXX C)

# Default to C11
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 11)
  set(CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -Wall -O3 -fPIC")
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
  set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -Wall --std=c++11 -O3 -fPIC")
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(CUDA REQUIRED)
find_package(ament_cmake REQUIRED)
find_package(rcutils REQUIRED)
find_package(rmw REQUIRED)
# find_package(rosidl_runtime_c REQUIRED)
# find_package(rosidl_typesupport_introspection_c REQUIRED)
# find_package(rosidl_typesupport_introspection_cpp REQUIRED)
# find_package(microcdr REQUIRED)

set(CUDA_SEPARABLE_COMPILATION ON)
cuda_select_nvcc_arch_flags(ARCH_FLAGS Auto)
set(CUDA_NVCC_FLAGS ${ARCH_FLAGS} CACHE STRING "nvcc flags" FORCE)

include_directories(include)

include_directories(${CUDA_INCLUDE_DIRS})

# TODO: Separate out different allocators by domain based on which ones need extra libraries
set(hazcat_allocator_sources
  src/hma_template.c
  src/cpu_ringbuf_allocator.c
  src/cuda_ringbuf_allocator.c
)

add_library(hazcat_allocators ${hazcat_allocator_sources})
target_include_directories(hazcat_allocators PUBLIC
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
  "$<INSTALL_INTERFACE:include>")
  target_link_libraries(hazcat_allocators cuda)
ament_target_dependencies(hazcat_allocators
  CUDA
  rcutils
)

install(
  TARGETS hazcat_allocators
  EXPORT hazcat_allocators
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin)

ament_export_include_directories(include)
ament_export_libraries(hazcat_allocators)
ament_export_targets(${PROJECT_NAME})

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)

  ament_lint_auto_find_test_dependencies()
  find_package(test_msgs REQUIRED)
  find_package(std_msgs REQUIRED)

  ament_add_gtest(allocator_cpu_test test/hazcat_cpu_allocator_test.cpp)
  target_link_libraries(allocator_cpu_test hazcat_allocators)
  ament_target_dependencies(allocator_cpu_test
    test_msgs std_msgs
    rcutils
  )

  ament_add_gtest(allocator_cuda_test test/hazcat_cuda_allocator_test.cpp)
  target_link_libraries(allocator_cuda_test hazcat_allocators)
  ament_target_dependencies(allocator_cuda_test
    test_msgs std_msgs
    rcutils
  )
endif()

ament_package()

install(
  DIRECTORY include/
  DESTINATION include
)
