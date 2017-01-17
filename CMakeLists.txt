cmake_minimum_required(VERSION 3.4)

project(nccl VERSION 1.3.2)

find_package(CUDA REQUIRED)

file(GLOB _cuda_src src/*.cu)
file(GLOB _cuda_hdr src/*.h)

add_definitions(-DWIN32_LEAN_AND_MEAN
                -DNOMINMAX
                -DNCCL_MAJOR=${PROJECT_VERSION_MAJOR}
                -DNCCL_MINOR=${PROJECT_VERSION_MINOR}
                -DNCCL_PATCH=${PROJECT_VERSION_PATCH}
                -DCUDA_MAJOR=${CUDA_VERSION_MAJOR}
                -DCUDA_MINOR=${CUDA_VERSION_MINOR}
                )
                
if(MSVC AND CMAKE_GENERATOR MATCHES Ninja)        
  foreach(_suffix "" ${CMAKE_CONFIGURATION_TYPES})
    if(NOT _suffix STREQUAL "")
      string(TOUPPER _${_suffix} _suffix)
    endif()
    set(CMAKE_CXX_FLAGS${_suffix} "${CMAKE_CXX_FLAGS${_suffix}} /FS")
    set(CMAKE_C_FLAGS${_suffix} "${CMAKE_C_FLAGS${_suffix}} /FS")              
  endforeach()
endif()                

cuda_add_library(nccl STATIC ${_cuda_src} ${_cuda_hdr})

install(TARGETS nccl
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        )
