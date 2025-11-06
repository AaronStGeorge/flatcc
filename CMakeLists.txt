# Copyright 2020 The IREE Authors
#
# Licensed under the Apache License v2.0 with LLVM Exceptions.
# See https://llvm.org/LICENSE.txt for license information.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception

# Simplified flatcc build for IREE integration using standard CMake.
# This file is designed to be upstreamable to flatcc.

cmake_minimum_required(VERSION 3.13)
project(flatcc C)

# Build options
option(FLATCC_INSTALL "Enable install targets" OFF)

# Get the root directory of flatcc sources
set(FLATCC_ROOT "${CMAKE_CURRENT_SOURCE_DIR}")

#-------------------------------------------------------------------------------
# flatcc::parsing - Minimal library with parsing and verification only
#-------------------------------------------------------------------------------

add_library(flatcc_parsing STATIC
  "${FLATCC_ROOT}/config/config.h"
  "${FLATCC_ROOT}/src/runtime/verifier.c"
)

target_include_directories(flatcc_parsing
  SYSTEM PUBLIC
    $<BUILD_INTERFACE:${FLATCC_ROOT}/include>
    $<INSTALL_INTERFACE:include>
)

# Suppress warnings from flatcc headers by using SYSTEM includes
set_target_properties(flatcc_parsing PROPERTIES
  C_STANDARD 11
  POSITION_INDEPENDENT_CODE ON
)

# Create namespace alias
add_library(flatcc::parsing ALIAS flatcc_parsing)

#-------------------------------------------------------------------------------
# flatcc::runtime - Full runtime library with builder, JSON support, etc.
#-------------------------------------------------------------------------------

add_library(flatcc_runtime STATIC
  "${FLATCC_ROOT}/src/runtime/builder.c"
  "${FLATCC_ROOT}/src/runtime/emitter.c"
  "${FLATCC_ROOT}/src/runtime/json_parser.c"
  "${FLATCC_ROOT}/src/runtime/json_printer.c"
  "${FLATCC_ROOT}/src/runtime/refmap.c"
)

target_include_directories(flatcc_runtime
  PUBLIC
    $<BUILD_INTERFACE:${FLATCC_ROOT}/include>
    $<INSTALL_INTERFACE:include>
)

target_link_libraries(flatcc_runtime
  PUBLIC
    flatcc_parsing
)

set_target_properties(flatcc_runtime PROPERTIES
  C_STANDARD 11
  POSITION_INDEPENDENT_CODE ON
)

# Create namespace alias
add_library(flatcc::runtime ALIAS flatcc_runtime)

#-------------------------------------------------------------------------------
# iree-flatcc-cli - Compiler executable for generating C code from .fbs schemas
#-------------------------------------------------------------------------------

add_executable(iree-flatcc-cli
  "${FLATCC_ROOT}/src/cli/flatcc_cli.c"
  "${FLATCC_ROOT}/external/hash/cmetrohash64.c"
  "${FLATCC_ROOT}/external/hash/str_set.c"
  "${FLATCC_ROOT}/external/hash/ptr_set.c"
  "${FLATCC_ROOT}/src/compiler/hash_tables/symbol_table.c"
  "${FLATCC_ROOT}/src/compiler/hash_tables/scope_table.c"
  "${FLATCC_ROOT}/src/compiler/hash_tables/name_table.c"
  "${FLATCC_ROOT}/src/compiler/hash_tables/schema_table.c"
  "${FLATCC_ROOT}/src/compiler/hash_tables/value_set.c"
  "${FLATCC_ROOT}/src/compiler/fileio.c"
  "${FLATCC_ROOT}/src/compiler/parser.c"
  "${FLATCC_ROOT}/src/compiler/semantics.c"
  "${FLATCC_ROOT}/src/compiler/coerce.c"
  "${FLATCC_ROOT}/src/compiler/codegen_schema.c"
  "${FLATCC_ROOT}/src/compiler/flatcc.c"
  "${FLATCC_ROOT}/src/compiler/codegen_c.c"
  "${FLATCC_ROOT}/src/compiler/codegen_c_reader.c"
  "${FLATCC_ROOT}/src/compiler/codegen_c_sort.c"
  "${FLATCC_ROOT}/src/compiler/codegen_c_builder.c"
  "${FLATCC_ROOT}/src/compiler/codegen_c_verifier.c"
  "${FLATCC_ROOT}/src/compiler/codegen_c_sorter.c"
  "${FLATCC_ROOT}/src/compiler/codegen_c_json_parser.c"
  "${FLATCC_ROOT}/src/compiler/codegen_c_json_printer.c"
  "${FLATCC_ROOT}/src/runtime/builder.c"
  "${FLATCC_ROOT}/src/runtime/emitter.c"
  "${FLATCC_ROOT}/src/runtime/refmap.c"
)

target_include_directories(iree-flatcc-cli
  SYSTEM PRIVATE
    "${FLATCC_ROOT}/external"
    "${FLATCC_ROOT}/include"
    "${FLATCC_ROOT}/config"
)

set_target_properties(iree-flatcc-cli PROPERTIES
  C_STANDARD 11
)

#-------------------------------------------------------------------------------
# Installation (optional)
#-------------------------------------------------------------------------------

if(FLATCC_INSTALL)
  include(GNUInstallDirs)

  install(
    TARGETS flatcc_parsing flatcc_runtime
    EXPORT flatccTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  )

  install(
    TARGETS iree-flatcc-cli
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    BUNDLE DESTINATION ${CMAKE_INSTALL_BINDIR}
  )

  install(
    DIRECTORY "${FLATCC_ROOT}/include/"
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    FILES_MATCHING PATTERN "*.h"
  )

  install(
    EXPORT flatccTargets
    FILE flatccTargets.cmake
    NAMESPACE flatcc::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/flatcc
  )
endif()