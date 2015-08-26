cmake_minimum_required(VERSION 3.0)

project(cis565_getting_started)

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})

# Set up include and lib paths
set(EXTERNAL "external")
include_directories("${EXTERNAL}/include")
include_directories("${EXTERNAL}/src")
if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    set(EXTERNAL_LIB_PATH "${EXTERNAL}/lib/osx")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    set(EXTERNAL_LIB_PATH "${EXTERNAL}/lib/linux" "/usr/lib64")
elseif(WIN32)
    set(EXTERNAL_LIB_PATH "${EXTERNAL}/lib/win")
endif()
link_directories(${EXTERNAL_LIB_PATH})
list(APPEND CMAKE_LIBRARY_PATH "${EXTERNAL_LIB_PATH}")

# Find up and set up core dependency libs

set(GLFW_INCLUDE_DIR "${EXTERNAL}/include")
set(GLFW_LIBRARY_DIR "${CMAKE_LIBRARY_PATH}")
find_library(GLFW_LIBRARY "glfw3" HINTS "${GLFW_LIBRARY_DIR}")

set(GLEW_INCLUDE_DIR "${EXTERNAL}/include")
set(GLEW_LIBRARY_DIR "${CMAKE_LIBRARY_PATH}")
add_definitions(-DGLEW_STATIC)
find_package(GLEW)

find_package(OpenGL)

set(CORELIBS
    "${GLFW_LIBRARY}"
    "${OPENGL_LIBRARY}"
    "${GLEW_LIBRARY}"
    )

# Compiler flags
set(CMAKE_CXX_STANDARD 11)

# OSX-specific hacks/fixes
if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    list(APPEND CORELIBS "-framework IOKit")
    list(APPEND CORELIBS "-framework Cocoa")
    list(APPEND CORELIBS "-framework CoreVideo")
    # Use C++11
    add_compile_options(-std=c++11)
    list(APPEND CUDA_NVCC_FLAGS "-std=c++11")
    # Link against libstdc++ since CUDA doesn't support libc++ yet
    add_compile_options(-stdlib=libstdc++)
    list(APPEND CMAKE_EXE_LINKER_FLAGS "-stdlib=libstdc++")
    list(APPEND CUDA_NVCC_FLAGS "--compiler-options;-stdlib=libstdc++")
endif()

# Linux-specific hacks/fixes
if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    list(APPEND CMAKE_EXE_LINKER_FLAGS "-lX11 -lXxf86vm -lXrandr -lpthread -lXi")
endif()

# Crucial magic for CUDA linking
find_package(Threads REQUIRED)
find_package(CUDA REQUIRED)

set(CUDA_ATTACH_VS_BUILD_RULE_TO_CUDA_FILE ON)
set(CUDA_SEPARABLE_COMPILATION ON)

if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    set(CUDA_PROPAGATE_HOST_FLAGS OFF)
endif()

add_subdirectory(src)

cuda_add_executable(${CMAKE_PROJECT_NAME}
    "src/main.hpp"
    "src/main.cpp"
    )

target_link_libraries(${CMAKE_PROJECT_NAME}
    src
    ${CORELIBS}
    )