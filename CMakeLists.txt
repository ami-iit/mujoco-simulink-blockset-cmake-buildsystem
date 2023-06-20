# Copyright (C) 2023 Fondazione Istituto Italiano di Tecnologia (IIT). All rights reserved.
# This software may be modified and distributed under the terms of the BSD-3-Clause license

cmake_minimum_required(VERSION 3.16)

set(MUJOCO_SIMULINK_BLOCKSET_UPSTREAM_VERSION 2.0)
set(MUJOCO_SIMULINK_BLOCKSET_CMAKE_REVISION 0)
set(MUJOCO_SIMULINK_BLOCKSET_CMAKE_VERSION "${MUJOCO_SIMULINK_BLOCKSET_UPSTREAM_VERSION}.${MUJOCO_SIMULINK_BLOCKSET_CMAKE_REVISION}")
project(mujoco-simulink-blockset-cmake-buildsystem
  LANGUAGES C CXX
  VERSION ${MUJOCO_SIMULINK_BLOCKSET_CMAKE_VERSION})

include(GNUInstallDirs)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")

option(BUILD_SHARED_LIBS "Build libraries as shared as opposed to static" ON)
option(BUILD_TESTING "Create tests using CMake" OFF)

if(BUILD_TESTING)
  enable_testing()
endif()

set(MUJOCO_SIMULINK_BLOCKSET_INSTALL_DIR "mex/mujoco_simulink_blockset" CACHE STRING 
    "Location where the library is installed")
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/ycm-0.15)

# Enable RPATH support for installed binaries and libraries
include(AddInstallRPATHSupport)
add_install_rpath_support(BIN_DIRS "${CMAKE_INSTALL_FULL_BINDIR}"
  LIB_DIRS "${CMAKE_INSTALL_FULL_LIBDIR}"
  INSTALL_NAME_DIR "${CMAKE_INSTALL_FULL_LIBDIR}"
  USE_LINK_PATH)

# Encourage user to specify a build type (e.g. Release, Debug, etc.), otherwise set it to Release.
if(NOT CMAKE_CONFIGURATION_TYPES)
  if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY VALUE "Release")
  endif()
endif()

## Find dependencies
find_package(mujoco REQUIRED)
find_package(glfw3 REQUIRED)

## Clone mujoco-simulink-blockset repository
## To use with some the mujoco-simulink-blockset source coming from some other source, comment out
# the FetchContent* commands and set manually mujoco-simulink-blockset_SOURCE_DIR
include(FetchContent)
FetchContent_Declare(
  mujoco-simulink-blockset
  GIT_REPOSITORY https://github.com/mathworks-robotics/mujoco-simulink-blockset
# GIT_TAG        v${MUJOCO_SIMULINK_BLOCKSET_UPSTREAM_VERSION}
  GIT_TAG        9d54a4b5b4e2550432828f716934119e968455b4
  )

FetchContent_MakeAvailable(mujoco-simulink-blockset)

find_package(Matlab REQUIRED COMPONENTS ENG_LIBRARY SIMULINK)

# Create and install mex files
function(mjb_build_and_install_mex mexname)
  matlab_add_mex(
    NAME ${mexname}
    OUTPUT_NAME ${mexname}
    SRC ${mujoco-simulink-blockset_SOURCE_DIR}/src/mj.cpp ${mujoco-simulink-blockset_SOURCE_DIR}/src/${mexname}.cpp
    NO_IMPLICIT_LINK_TO_MATLAB_LIBRARIES
    LINK_TO ${Matlab_MEX_LIBRARY} ${Matlab_MX_LIBRARY} ${Matlab_DATAARRAY_LIBRARY} glfw mujoco::mujoco)
  target_compile_features(${mexname} PRIVATE cxx_std_17)
  install(
    TARGETS ${mexname}
    EXPORT ${PROJECT_NAME}
    DESTINATION ${MUJOCO_SIMULINK_BLOCKSET_INSTALL_DIR})
endfunction()

mjb_build_and_install_mex(mj_depth_near_far)
mjb_build_and_install_mex(mj_initbus_mex)
mjb_build_and_install_mex(mj_sampletime)
mjb_build_and_install_mex(mj_sfun)

# Install block files
file(GLOB block_files ${mujoco-simulink-blockset_SOURCE_DIR}/blocks/*.m 
                      ${mujoco-simulink-blockset_SOURCE_DIR}/blocks/*.xml 
                      ${mujoco-simulink-blockset_SOURCE_DIR}/blocks/*.slx)
install(FILES ${block_files} DESTINATION ${MUJOCO_SIMULINK_BLOCKSET_INSTALL_DIR})

# Install examples files
file(GLOB example_files ${mujoco-simulink-blockset_SOURCE_DIR}/examples/*.slx)
install(FILES ${example_files} DESTINATION ${MUJOCO_SIMULINK_BLOCKSET_INSTALL_DIR})

include(AddUninstallTarget)
