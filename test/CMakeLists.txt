include_directories(include)

file(GLOB _single_tests single/*.cu)

# TODO move find nvtoolsext to a find module
if(CMAKE_CL_64)
  set(_arch 64)
  set(_cuda_arch x64)
else()
  set(_arch 32)
  set(_cuda_arch Win32)
endif()

find_path(NVTOOLSEXT_INCLUDE_DIR nvToolsExt.h
          PATH_SUFFIXES "NVIDIA Corporation/NvToolsExt/include")

find_library(NVTOOLSEXT_LIBRARY nvToolsExt${_arch}_1
          PATH_SUFFIXES "NVIDIA Corporation/NvToolsExt/lib/${_cuda_arch}")

find_file(NVTOOLSEXT_SHARED_LIBRARY nvToolsExt${_arch}_1.dll
          PATH_SUFFIXES "NVIDIA Corporation/NvToolsExt/bin/${_cuda_arch}")

find_file(NVML_SHARED_LIBRARY nvml.dll
          PATH_SUFFIXES "NVIDIA Corporation/NVSMI")


if(NOT NVTOOLSEXT_INCLUDE_DIR OR
   NOT NVTOOLSEXT_LIBRARY OR
   NOT NVTOOLSEXT_SHARED_LIBRARY)
    message(FATAL_ERROR "Could not find nvtoolsext")
endif()

if(NOT NVML_SHARED_LIBRARY)
    message(FATAL_ERROR "Could not find nvml.dll")
endif()

include_directories(${NVTOOLSEXT_INCLUDE_DIR})

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/copy_nv_shared_libs.stamp
                   COMMAND ${CMAKE_COMMAND} -E copy_if_different ${NVTOOLSEXT_SHARED_LIBRARY} ${CMAKE_CURRENT_BINARY_DIR}
                   COMMAND ${CMAKE_COMMAND} -E copy_if_different ${NVML_SHARED_LIBRARY} ${CMAKE_CURRENT_BINARY_DIR}
                   COMMAND ${CMAKE_COMMAND} -E touch ${CMAKE_CURRENT_BINARY_DIR}/copy_nv_shared_libs.stamp
                  )

add_custom_target(copy_nv_shared_libs ALL DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/copy_nv_shared_libs.stamp)

foreach(_single_test ${_single_tests})
  get_filename_component(_single_test_name ${_single_test} NAME_WE)
  set(_target_name ${_single_test_name})
  cuda_add_executable(${_target_name} ${_single_test} ${CMAKE_CURRENT_BINARY_DIR}/copy_nv_shared_libs.stamp)
  add_dependencies(${_target_name} copy_nv_shared_libs)
  target_link_libraries(${_target_name} nccl ${NVTOOLSEXT_LIBRARY} ${CUDA_curand_LIBRARY})
endforeach()
