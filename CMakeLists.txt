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

find_package(ament_cmake REQUIRED)
find_package(PkgConfig REQUIRED)
include(FetchContent)
include(ExternalProject)

# Fetch MOSEK Optimization Suite
#---------------------------------
fetchcontent_declare(
  mosek_binaries
  URL      https://download.mosek.com/stable/${MOSEK_VERSION}/mosektoolslinux64x86.tar.bz2
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
  ${mosek_binaries_SOURCE_DIR}/${VERSION_MOSEK_MAJOR}.${VERSION_MOSEK_MINOR}/tools/platform/linux64x86/
)

set(MOSEK_HEADERDIR ${MOSEK_SRC_LINUX}/h)
set(MOSEK_SRCDIR ${MOSEK_SRC_LINUX}/src/fusion_cxx)
set(MOSEK_LIBDIR ${MOSEK_SRC_LINUX}/bin)


# Build the C++ API (Fusion)
#-------------------------------

# message("Building the C++ API (Fusion)")
execute_process(
  WORKING_DIRECTORY ${MOSEK_SRCDIR}
  COMMAND make install
)
set(CMAKE_VERBOSE_MAKEFILE ON)

#[==[
set(CMAKE_VERBOSE_MAKEFILE ON)

ExternalProject_Add(fusion_lib
	SOURCE_DIR ${MOSEK_SRCDIR}
  # BINARY_DIR ${MOSEK_LIBDIR}
  INSTALL_DIR ${MOSEK_LIBDIR}
	CONFIGURE_COMMAND ""
	BUILD_IN_SOURCE ON  # use the existing Makefile
)
#]==]

# Setup vendor library
#----------------------

add_library(${PROJECT_NAME} INTERFACE)
target_include_directories(${PROJECT_NAME} INTERFACE
  $<BUILD_INTERFACE:${MOSEK_HEADERDIR}>
  $<INSTALL_INTERFACE:include>
)

target_link_libraries(
  ${PROJECT_NAME}
  INTERFACE
    ${CMAKE_INSTALL_PREFIX}/lib/libmosek64.so.${VERSION_MOSEK_MAJOR}.${VERSION_MOSEK_MINOR}
    ${CMAKE_INSTALL_PREFIX}/lib/libfusion64.so.${VERSION_MOSEK_MAJOR}.${VERSION_MOSEK_MINOR}
)

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

install(
  DIRECTORY ${MOSEK_HEADERDIR}/
  DESTINATION ./include
)

install(
  FILES ${MOSEK_LIBDIR}/libmosek64.so.${VERSION_MOSEK_MAJOR}.${VERSION_MOSEK_MINOR}
  DESTINATION lib
  # RENAME libdhd.so.${VERSION_SDK_MAJOR}
)
install(
  FILES ${MOSEK_LIBDIR}/libfusion64.so.${VERSION_MOSEK_MAJOR}.${VERSION_MOSEK_MINOR}
  DESTINATION lib
  # RENAME libdrd.so.${VERSION_SDK_MAJOR}
)
install(
  TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
)

ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)

ament_package()
