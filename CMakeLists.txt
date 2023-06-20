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

set(OSQP_MATLAB_INSTALL_MATLAB_LIBDIR "mex" CACHE
    STRING "Location (relative to the install prefix) in which the Matlab mex libraries are installed.")
set(OSQP_MATLAB_INSTALL_MATLAB_MFILESDIR "mex" CACHE
    STRING "Location (relative to the install prefix) in which the Matlab .m files are installed.")

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

# Common source files
set(M_FILES ${osqp-matlab_SOURCE_DIR}/osqp.m)
set(MEX_FILES ${osqp-matlab_SOURCE_DIR}/osqp_mex.hpp ${osqp-matlab_SOURCE_DIR}/osqp_mex.cpp)

if(OSQP_MATLAB_USES_MATLAB)
  find_package(Matlab REQUIRED)
  matlab_add_mex(
      NAME osqp_mex_matlab
      OUTPUT_NAME osqp_mex
      SRC ${MEX_FILES}
      NO_IMPLICIT_LINK_TO_MATLAB_LIBRARIES
      LINK_TO ${Matlab_MEX_LIBRARY} ${Matlab_MX_LIBRARY} qdldl::qdldl osqp::osqp)
  # Workaround for missing include https://github.com/oxfordcontrol/osqp-matlab/issues/27
  target_include_directories(osqp_mex_matlab PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/workaround_include)
  install(
    TARGETS osqp_mex_matlab
    EXPORT ${PROJECT_NAME}
    DESTINATION ${OSQP_MATLAB_INSTALL_MATLAB_LIBDIR})
  install(
    FILES ${M_FILES}
    DESTINATION ${OSQP_MATLAB_INSTALL_MATLAB_MFILESDIR})

    # Enable tests
    if (BUILD_TESTING)
      find_package(Matlab REQUIRED
               COMPONENTS MAIN_PROGRAM)
      configure_file(${CMAKE_CURRENT_SOURCE_DIR}/run_osqp_tests_no_codegen.m.in ${CMAKE_CURRENT_BINARY_DIR}/run_osqp_tests_no_codegen.m @ONLY)
      # Note: this only works if the project was installed in the correct location and can be found with MATLABPATH
      add_test(NAME matlab_osqp_tests
           COMMAND ${Matlab_MAIN_PROGRAM} -nodisplay -nodesktop -nojvm -batch "addpath('${CMAKE_CURRENT_BINARY_DIR}');run_osqp_tests_no_codegen;")

    endif()
endif()

include(AddUninstallTarget)
