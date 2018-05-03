###############################################################################
# Copyright (c) 2014, Jefferson Amstutz
# Copyright (c) 2014, SURVICE Engineering
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are
# met:
#
#   * Redistributions of source code must retain the above copyright
#     notice, this list of conditions and the following disclaimer.
#
#   * Redistributions in binary form must reproduce the above copyright
#     notice, this list of conditions and the following disclaimer in
#     the documentation and/or other materials provided with the
#     distribution.
#
#   * Neither the name of SURVICE Engineering nor the names of its
#     contributors may be used to endorse or promote products derived
#     from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
# A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
# LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
###############################################################################

project(ftevaluator)
cmake_minimum_required(VERSION 2.8)

##############################################################
# ISA configuration
##############################################################

set(XEON_ISA "AVX" CACHE STRING "Select highest ISA to support (SSE2, SSE3, SSSE3, SSE4.1, SSE4.2, AVX, AVX-I, AVX2")

if (XEON_ISA STREQUAL "SSE2")
  set(ISA  1)
endif ()

if (XEON_ISA STREQUAL "SSE3")
  set(ISA  2)
endif ()

if (XEON_ISA STREQUAL "SSSE3")
  set(ISA  3)
endif ()

if (XEON_ISA STREQUAL "SSE4.1")
  set(ISA  4)
endif ()

if (XEON_ISA STREQUAL "SSE4.2")
  set(ISA  5)
endif ()

if (XEON_ISA STREQUAL "AVX")
  set(ISA  6)
endif ()

if (XEON_ISA STREQUAL "AVX-I")
  set(ISA  7)
endif ()

if (XEON_ISA STREQUAL "AVX2")
  set(ISA  8)
endif ()

set(TARGET_SSE2  OFF)
set(TARGET_SSE3  OFF)
set(TARGET_SSSE3  OFF)
set(TARGET_SSE41  OFF)
set(TARGET_SSE42  OFF)
set(TARGET_AVX  OFF)
set(TARGET_AVXI  OFF)
set(TARGET_AVX2  OFF)

if (ISA GREATER 0)
  set(TARGET_SSE2  ON)
  set(ISPC_TARGETS "sse2")
endif ()

if (ISA GREATER 1)
  set(TARGET_SSE3  ON)
endif ()

if (ISA GREATER 2)
  set(TARGET_SSSE3  ON)
endif ()

if (ISA GREATER 3)
  set(TARGET_SSE41  ON)
endif ()

if (ISA GREATER 4)
  set(TARGET_SSE42  ON)
  set(ISPC_TARGETS ${ISPC_TARGETS} "sse4")
endif ()

if (ISA GREATER 5)
  set(TARGET_AVX  ON)
  set(ISPC_TARGETS ${ISPC_TARGETS} "avx")
endif ()

if (ISA GREATER 6)
  set(TARGET_AVXI  ON)
endif ()

if (ISA GREATER 7)
  set(TARGET_AVX2  ON)
  set(ISPC_TARGETS ${ISPC_TARGETS} "avx2")
endif ()

set(ISPC_TARGETS "${ISPC_TARGETS}")
string(REGEX REPLACE ";" "," ISPC_TARGETS "${ISPC_TARGETS}")

##############################################################
# Compiler
##############################################################

if(${CMAKE_CXX_COMPILER_ID} STREQUAL "Intel")
  include(cmake/icc.cmake)
elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU")
  include(cmake/gcc.cmake)
elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL "Clang")
  include(cmake/clang.cmake)
elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL "MSVC")
  include(cmake/msvc.cmake)
else()
  message(FATAL_ERROR "Unsupported compiler: '${CMAKE_CXX_COMPILER_ID}'")
endif()

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING
      "Choose the build type: Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif(NOT CMAKE_BUILD_TYPE)

set(CMAKE_VERBOSE_MAKEFILE false CACHE BOOL "Enables verbose mode.")

set(__XEON__ 1)

include(cmake/ispc.cmake)

##############################################################
# Application
##############################################################

set(FTE_THREADING ON CACHE BOOL "Enable OpenMP threading for evaluations")

if(FTE_THREADING)
  add_definitions(-DEVAL_THREADING)
endif()

set(FTE_ISPC_PACKET_SIZE 16 CACHE INT "ISPC evaluation packet size")

add_definitions(-DPACKET_SIZE=${FTE_ISPC_PACKET_SIZE})

set(FTE_ISPC_EVAL_ENABLED ON CACHE BOOL "Use ISPC code path when evaluating")

if(FTE_ISPC_EVAL_ENABLED)
  add_definitions(-DUSE_ISPC)
endif()

set(FTE_INIT_COMPONENTS OFF CACHE BOOL "Initialize component values to [0.0,1.0)")

if(FTE_INIT_COMPONENTS)
  add_definitions(-DINIT_COMPONENTS)
endif()

set(FTE_EVAL_CHUNK_SIZE 4096 CACHE INT "Maximum size of components buffer in-flight (in MB)")

add_definitions(-DCHUNK_SIZE=${FTE_EVAL_CHUNK_SIZE})

include_directories(${CMAKE_SOURCE_DIR})

set(fte_SRCS
  ftParser.h
  ftEvaluator.h
  ftTypes.h

  ftEvaluator.cpp
  ftEvaluator.ispc
  ftParser.cpp
  ftNode.cpp
  ftNodeVisitor.cpp
  ftFileReader.cpp
  ftNamePool.cpp
)

add_ispc_executable(fte ${fte_SRCS} main.cpp)

###############################################################################
# Add 'make install' target

install(TARGETS ${EXEC_NAME} RUNTIME DESTINATION bin)

