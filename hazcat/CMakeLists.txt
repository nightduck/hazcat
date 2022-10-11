cmake_minimum_required(VERSION 3.8)
set(CMAKE_CUDA_COMPILER /usr/local/cuda/bin/nvcc)
project(hazcat CUDA CXX C)

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
find_package(hazcat_allocators REQUIRED)
# find_package(rosidl_runtime_c REQUIRED)
# find_package(rosidl_typesupport_introspection_c REQUIRED)
# find_package(rosidl_typesupport_introspection_cpp REQUIRED)
# find_package(microcdr REQUIRED)

set(CUDA_SEPARABLE_COMPILATION ON)
cuda_select_nvcc_arch_flags(ARCH_FLAGS Auto)
set(CUDA_NVCC_FLAGS ${ARCH_FLAGS} CACHE STRING "nvcc flags" FORCE)

include_directories(include)

include_directories(${CUDA_INCLUDE_DIRS})

set(hazcat_message_queue_sources
  src/hazcat_message_queue.c
  src/guard_condition_impl.c
)

add_library(hazcat ${hazcat_message_queue_sources})
target_include_directories(hazcat PUBLIC
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
  "$<INSTALL_INTERFACE:include>")
ament_target_dependencies(hazcat
  rmw
)

install(
  TARGETS hazcat
  EXPORT hazcat
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin)

ament_export_include_directories(include)
ament_export_libraries(hazcat)
ament_export_targets(${PROJECT_NAME})

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)

  ament_lint_auto_find_test_dependencies()
endif()

ament_package()

install(
  DIRECTORY include/
  DESTINATION include
)
