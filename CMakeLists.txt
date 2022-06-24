cmake_minimum_required(VERSION 3.17 FATAL_ERROR)

project(
  vector_add_example
  DESCRIPTION "Example using C++ Wrappers for the CUDA Driver API"
  VERSION 0.0.1
  HOMEPAGE_URL https://github.com/nlesc-recruit/usage-example-git-submodules
  LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_BUILD_TYPE Release)

# Use CMAKE_INSTALL_PREFIX when searching for libraries to run the installed
# vector_add_example program.
include(GNUInstallDirs)
list(APPEND CMAKE_INSTALL_RPATH ${CMAKE_INSTALL_FULL_LIBDIR})

find_package(cudawrappers 0.3.0 REQUIRED cu nvrtc)

add_executable(vector_add_example src/vector_add.cpp)

target_link_libraries(vector_add_example PRIVATE cudawrappers::cu
                                                 cudawrappers::nvrtc)

# Make it possible to #include cuda source code
function(include_cuda_code target input_file)
  # Save file containing cuda code as a C++ raw string literal
  file(READ ${input_file} content)
  set(delim "for_c++_include")
  set(content "R\"${delim}(\n${content})${delim}\"")
  set(output_file "${CMAKE_CURRENT_BINARY_DIR}/${input_file}")
  file(WRITE ${output_file} "${content}")
  # Add save path to the include directories
  get_filename_component(output_subdir ${input_file} DIRECTORY)
  target_include_directories(
    ${target} PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/${output_subdir}")
endfunction(include_cuda_code)

include_cuda_code(vector_add_example src/kernels/vector_add_kernel.cu)

install(TARGETS vector_add_example)
