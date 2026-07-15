cmake_minimum_required(VERSION 3.27)
project(ps3ldr LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(REPO_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/../..")
set(ELF_COMMON_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../elf_common")
set(THIRD_PARTY_PATH ${CMAKE_SOURCE_DIR}/../../third_party)
set(IDA_SDK_ROOT "${REPO_ROOT}/third_party/ida-sdk/src")
set(IDA_INCLUDE_DIR "${IDA_SDK_ROOT}/include")
set(IDA_LOADER_DIR "${IDA_SDK_ROOT}/ldr")
set(IDA_LIBRARY "${IDA_SDK_ROOT}/lib/x64_win_64/ida.lib")

if(NOT EXISTS "${IDA_INCLUDE_DIR}/ida.hpp")
    message(FATAL_ERROR
        "The ida-sdk submodule was not found at ${IDA_SDK_ROOT}. "
        "Initialize submodules before configuring this project.")
endif()

if(NOT EXISTS "${IDA_LIBRARY}")
    message(FATAL_ERROR
        "The 64-bit Windows IDA import library was not found at ${IDA_LIBRARY}.")
endif()

if(NOT MSVC)
    message(FATAL_ERROR "The PS3 loader is configured for Visual Studio/MSVC builds.")
endif()

if(NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
    message(FATAL_ERROR "Build the PS3 loader with a 64-bit generator (-A x64).")
endif()

set(SOURCES
    "${ELF_COMMON_PATH}/elf_reader.h"
    "${ELF_COMMON_PATH}/elf.h"
    ${THIRD_PARTY_PATH}/tinyxml/tinyxml2.cpp
    ${THIRD_PARTY_PATH}/tinyxml/tinyxml2.h
    cell_loader.cpp
    cell_loader.h
    ps3.cpp
    sce.h
)

add_library(ps3ldr SHARED ${SOURCES})

target_include_directories(ps3ldr PRIVATE
    "${IDA_INCLUDE_DIR}"
    "${IDA_LOADER_DIR}"
    "${ELF_COMMON_PATH}"
    "${THIRD_PARTY_PATH}/tinyxml"
)

target_compile_definitions(ps3ldr PRIVATE
    __IDP__
    __EA64__
)

target_link_libraries(ps3ldr PRIVATE "${IDA_LIBRARY}")

set_target_properties(ps3ldr PROPERTIES
    MSVC_RUNTIME_LIBRARY "MultiThreadedDLL"
    OUTPUT_NAME "ps3"
    PREFIX ""
    SUFFIX ".dll"
)