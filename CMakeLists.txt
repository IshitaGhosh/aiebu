# SPDX-License-Identifier: MIT
# Copyright (C) 2024-2025 Advanced Micro Devices, Inc. All rights reserved.
cmake_minimum_required(VERSION 3.18.0)
project(AIEBU HOMEPAGE_URL https://github.com/Xilinx/aiebu VERSION 1.0)

include(cmake/settings.cmake)

message("-- CMAKE_SYSTEM=${CMAKE_SYSTEM}")
message("-- CMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}")

# These variables may be defined by the parent project as it may also include
# AIE-RT and or ELFIO as a submodule
# AIEBU_AIE_RT_BIN_DIR, AIEBU_AIE_RT_HEADER_DIR and AIEBU_ELFIO_SRC_DIR

if (NOT (DEFINED AIEBU_AIE_RT_BIN_DIR))
  set(AIEBU_AIE_RT_BIN_DIR ${AIEBU_BINARY_DIR})
endif()

if (NOT (DEFINED AIEBU_AIE_RT_HEADER_DIR))
	set(AIEBU_AIE_RT_HEADER_DIR "${CMAKE_SOURCE_DIR}/lib/aie-rt/driver/driver-src/include")
  #  set(AIEBU_AIE_RT_HEADER_DIR "${AIEBU_BINARY_DIR}/lib/aie-rt/driver/driver-src/include")
endif()

message("-- Using aie-rt headers from ${AIEBU_AIE_RT_HEADER_DIR}")
#message("-- Using aie-rt build from ${AIEBU_AIE_RT_BIN_DIR}")

if (NOT (DEFINED AIEBU_ELFIO_SRC_DIR))
  set(AIEBU_ELFIO_SRC_DIR "${AIEBU_SOURCE_DIR}/src/cpp/ELFIO")
endif()

message("-- Using ELFIO from ${AIEBU_ELFIO_SRC_DIR}")

if (AIEBU_FULL STREQUAL "ON")
  add_compile_options(-DAIEBU_FULL)

  find_program (PYLINT pylint REQUIRED)
  message("-- Pylint path: ${PYLINT}")

  find_package(Python3 COMPONENTS Interpreter REQUIRED)
  message("-- Python version: ${Python3_VERSION}")
endif()

if (${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
  find_package(PkgConfig REQUIRED)

  pkg_check_modules(LIBELF REQUIRED libelf)
  message("-- Libelf version: ${LIBELF_VERSION}")

  if (AIEBU_FULL STREQUAL "ON")
    find_program (READELF eu-readelf REQUIRED)
    message("-- Readelf path: ${READELF}")
  endif()
endif()


if (XRT_CLANG_TIDY STREQUAL "ON")
  find_program(CLANG_TIDY "clang-tidy")
  if(NOT CLANG_TIDY)
    message("clang-tidy not found, cannot enable static analysis")
  else()
    message("-- Enabling clang-tidy")
    set(CMAKE_CXX_CLANG_TIDY "clang-tidy")
  endif()
endif()

add_subdirectory(src/cpp)

if (AIEBU_FULL STREQUAL "ON")
  add_subdirectory(src/python)
  add_subdirectory(specification)
  add_subdirectory(scripts)
endif()

add_subdirectory(lib)

add_subdirectory(specification)

add_subdirectory(lib)
#add_subdirectory(test)

SET(PACKAGE_KIND "TGZ")
SET(CPACK_GENERATOR "TGZ")
message("-- ${CMAKE_BUILD_TYPE} ${PACKAGE_KIND} package")

SET(CPACK_PACKAGE_VENDOR "Advanced Micro Devices Inc.")
SET(CPACK_PACKAGE_CONTACT "runtimeca39d@amd.com")
SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY "AMD XDNA binutils package")
SET(CPACK_RESOURCE_FILE_LICENSE "${AIEBU_SOURCE_DIR}/LICENSE")

include (cmake/findpackage.cmake)

INCLUDE(CPack)

include(cmake/cpack.cmake)

