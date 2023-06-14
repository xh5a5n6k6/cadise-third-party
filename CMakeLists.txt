# CMake script to build all third-party libraries used in Cadise Renderer.

cmake_minimum_required(VERSION 3.21)
project(CadiseThirdParty)

# Restrict project to only be built from a 64-bit system.
if(NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
    message(FATAL_ERROR
        "This project requires a 64-bit system.")
endif()

#------------------------------------------------------------------------------
# OS/Compiler specific setup
#------------------------------------------------------------------------------
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
    set(BUILD_OS_TYPE "Windows")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    set(BUILD_OS_TYPE "Linux")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    set(BUILD_OS_TYPE "macOS")
else()
    message(FATAL_ERROR
        "Unknown OS type: ${CMAKE_SYSTEM_NAME}")
endif()

# Set OS-dependent install directory.
set(THIRD_PARTY_INSTALL_DIR 
    "${PROJECT_BINARY_DIR}/ThirdParty/${BUILD_OS_TYPE}/")

# Use C++20 to build all libraries.
set(CMAKE_CXX_STANDARD          20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /std:c++20")

    # Set multi-thread compilation
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")

    # Set debug information format
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Zi")

elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++20")
    
# Match for Clang or AppleClang
elseif(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++20")

else()
    message(STATUS 
        "Unknown compiler ID: ${CMAKE_CXX_COMPILER_ID}, no extra flag is set.")

endif()

#------------------------------------------------------------------------------
# Third-party libraries setup
#------------------------------------------------------------------------------
include(ExternalProject)

set(STACKWALKER_LIB_NAME "StackWalker")

#------------------------------------------------------------------------------
# StackWalker
#------------------------------------------------------------------------------
# Maybe replace it with the stacktrace library in C++23?
if(BUILD_OS_TYPE MATCHES "Windows")
    ExternalProject_Add(${STACKWALKER_LIB_NAME}
        # Directory option
        PREFIX      "${PROJECT_BINARY_DIR}/${STACKWALKER_LIB_NAME}/"
        SOURCE_DIR  "${PROJECT_SOURCE_DIR}/${STACKWALKER_LIB_NAME}/"
        BINARY_DIR  "${PROJECT_BINARY_DIR}/${STACKWALKER_LIB_NAME}/"
        INSTALL_DIR "${THIRD_PARTY_INSTALL_DIR}/${STACKWALKER_LIB_NAME}/"
    
        # Configure option
        CMAKE_CACHE_ARGS
            -DCMAKE_CXX_STANDARD:STRING=${CMAKE_CXX_STANDARD}
            -DCMAKE_CXX_STANDARD_REQUIRED:BOOL=ON
            -DCMAKE_CXX_FLAGS:STRING=${CMAKE_CXX_FLAGS}
            -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>

        # Build option
        BUILD_ALWAYS ON
    
        # Install option
        INSTALL_COMMAND ${CMAKE_COMMAND} --install <BINARY_DIR> --config $<CONFIG> --prefix <INSTALL_DIR>)
endif()