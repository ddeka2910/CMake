cmake_minimum_required(VERSION 3.18)
project(SharedRuntimePlusToolkit CXX)

#Goal for this example:
# Validate that with c++ we can use some components of the CUDA toolkit, and
# specify the cuda runtime
find_package(CUDAToolkit REQUIRED)

add_library(Common OBJECT curand.cpp nppif.cpp)
target_link_libraries(Common PRIVATE CUDA::toolkit)
set_target_properties(Common PROPERTIES POSITION_INDEPENDENT_CODE ON)

#shared runtime with shared toolkit libraries
add_library(SharedToolkit SHARED shared.cpp)
target_link_libraries(SharedToolkit PRIVATE Common PUBLIC CUDA::curand CUDA::nppif)
target_link_libraries(SharedToolkit PUBLIC CUDA::cudart)

# The CUDA only ships the shared version of the toolkit libraries
# on windows
if(NOT WIN32)
  #shared runtime with static toolkit libraries
  add_library(StaticToolkit SHARED static.cpp)
  target_link_libraries(StaticToolkit PRIVATE Common CUDA::curand_static CUDA::nppif_static)
  target_link_libraries(StaticToolkit PUBLIC CUDA::cudart)

  #static runtime with mixed toolkit libraries
  add_library(MixedToolkit SHARED mixed.cpp)
  target_link_libraries(MixedToolkit PRIVATE Common CUDA::curand_static CUDA::nppif)
  target_link_libraries(MixedToolkit PUBLIC CUDA::cudart)
endif()

add_executable(SharedRuntimePlusToolkit main.cpp)
target_link_libraries(SharedRuntimePlusToolkit PRIVATE SharedToolkit
                      $<TARGET_NAME_IF_EXISTS:StaticToolkit>
                      $<TARGET_NAME_IF_EXISTS:MixedToolkit>)
