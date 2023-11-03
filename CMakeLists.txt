# Copyright 2023 ICUBE Laboratory, University of Strasbourg
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# Author: Thibault Poignonec (thibault.poignonec@gmail.com)

cmake_minimum_required(VERSION 3.11)
project(mosek_vendor_ros2)
set(MOSEK_VERSION "10.1.16")

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17) # Default to C++17
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

find_package(PkgConfig REQUIRED)
pkg_check_modules(libusb REQUIRED libusb-1.0)

find_package(ament_cmake REQUIRED)
include(FetchContent)
include(ExternalProject)

# Fetch MOSEK Optimization Suite
#---------------------------------
fetchcontent_declare(
  mosek_binaries
  URL      https://download.mosek.com/stable/${MOSEK_VERSION}/mosektoolslinuxaarch64.tar.bz2
  INSTALL_COMMAND ""
)
fetchcontent_makeavailable(mosek_binaries)

# Set path variables
#-------------------------------
string(REPLACE "." ";" VERSION_MOSEK_LIST ${MOSEK_VERSION})
list(GET VERSION_MOSEK_LIST 0 VERSION_MOSEK_MAJOR)
list(GET VERSION_MOSEK_LIST 1 VERSION_MOSEK_MINOR)

set(
  MOSEK_SRC_LINUX
  ${mosek_binaries_SOURCE_DIR}/${VERSION_MOSEK_MAJOR}.${VERSION_MOSEK_MINOR}/tools/platform/linuxaarch64/
)

set(MOSEK_HEADERDIR ${MOSEK_SRC_LINUX}/h)
set(MOSEK_SRCDIR ${MOSEK_SRC_LINUX}/src/fusion_cxx)
set(MOSEK_LIBDIR ${MOSEK_SRC_LINUX}/bin)


# Build the C++ API (Fusion)
#-------------------------------

message("Building the C++ API (Fusion)")
execute_process(
  WORKING_DIRECTORY ${MOSEK_SRCDIR}
  COMMAND make install
)

#[==[
set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_INSTALL_RPATH ${MOSEK_LIBDIR})
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

list(APPEND CMAKE_PREFIX_PATH ${MOSEK_LIBDIR})

# list( APPEND CMAKE_INSTALL_RPATH ${MOSEK_LIBDIR} )
ExternalProject_Add(fusion_lib
	SOURCE_DIR ${MOSEK_SRCDIR}
  # BINARY_DIR ${MOSEK_LIBDIR}
  CMAKE_ARGS -DCMAKE_CXX_FLAGS:STRING=LDFLAGS=-L${MOSEK_LIBDIR}
  INSTALL_DIR ${MOSEK_LIBDIR}
	CONFIGURE_COMMAND ""
  CMAKE_ARGS 
	BUILD_IN_SOURCE ON  # use the existing Makefile
)
#]==]

# Setup vendor library
#----------------------

#[==[
add_library(${PROJECT_NAME} INTERFACE)
target_include_directories(${PROJECT_NAME} INTERFACE
  $<BUILD_INTERFACE:${fd_sdk_binaries_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)
target_link_libraries(
  ${PROJECT_NAME} INTERFACE
  ${CMAKE_INSTALL_PREFIX}/lib/libdhd.so.${VERSION_SDK_MAJOR}
  ${CMAKE_INSTALL_PREFIX}/lib/libdrd.so.${VERSION_SDK_MAJOR}
  ${libusb-1.0_LIBRARIES}
)
#]==]

# Testing
#----------
if(BUILD_TESTING)
  find_package(ament_cmake_test REQUIRED)
  find_package(ament_lint_auto REQUIRED)
  set(ament_cmake_copyright_FOUND TRUE)
  set(ament_cmake_cpplint_FOUND TRUE)

  ament_lint_auto_find_test_dependencies()
endif()

# Install
#----------

ament_package()
