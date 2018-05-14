PROJECT(bubbleRob_test LANGUAGES C CXX)
set(PROJECT_TEST_NAME ${PROJECT_NAME}_test)

# CMake project configuration
################################################################################
CMAKE_MINIMUM_REQUIRED(VERSION 3.3)

# CMake Policies
# Warning: Some of these differ from the original MRPT project
cmake_policy(SET CMP0003 NEW) # We don't want to mix relative and absolute paths in linker lib lists.
cmake_policy(SET CMP0005 NEW) # Escape definitions (-D) strings
cmake_policy(SET CMP0053 NEW) # Simplify variable reference and escape sequence evaluation.
cmake_policy(SET CMP0037 NEW)
cmake_policy(SET CMP0043 NEW)
cmake_policy(SET CMP0048 NEW)

# Package version management
set(PROJECT_VERSION_MAJOR 0)
set(PROJECT_VERSION_MINOR 1)
set(PROJECT_VERSION_PATCH 0)
set(PROJECT_VERSION_TWEAK 0)
string(CONCAT PROJECT_VERSION
    "${PROJECT_VERSION_MAJOR}."
    "${PROJECT_VERSION_MINOR}."
    "${PROJECT_VERSION_PATCH}."
    "${PROJECT_VERSION_TWEAK}")
message(STATUS "${PROJECT_NAME} Version: ${PROJECT_VERSION}")

# Where to place compilation targets
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(test_output_directory ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/test)

# Minimum compiler requirements
# same as MRPT-2.X
set(MIN_GCC_CXX_COMPILER_VERSION "7.0.0")
set(MIN_CLANG_CXX_COMPILER_VERSION "4.0.0")

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    if (NOT "${CMAKE_CXX_COMPILER_VERSION}" VERSION_GREATER
            ${MIN_CLANG_CXX_COMPILER_VERSION})
    endif()
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    if (NOT "${CMAKE_CXX_COMPILER_VERSION}" VERSION_GREATER
            ${MIN_GCC_CXX_COMPILER_VERSION})
    endif()
else()
    message(WARNING
        "Compiler \"${CMAKE_CXX_COMPILER_ID}\" is not yet supported by ${PROJECT_NAME}")
endif()


set(PROJECT_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../)

#V-REP Config
################################################################################

if (EXISTS "${PROJECT_ROOT_DIR}/../V-REP")
  message(STATUS "Found V-REP")
  set (V-REP_ROOT "${PROJECT_ROOT_DIR}/../" CACHE "V-REP root dir" STRING)

  execute_process (COMMAND "/bin/bash" "-c" "cd ${V-REP_ROOT}; echo -n `pwd`" OUTPUT_VARIABLE V-REP_ROOT)


  set (V-REP_PATH "${PROJECT_ROOT_DIR}/../V-REP/programming/" CACHE "The V-REP build path" STRING)

  set (SOURCES bubbleRob_test.cpp
    ${V-REP_ROOT}/V-REP/programming/remoteApi/extApi.c
    ${V-REP_ROOT}/V-REP/programming/common/shared_memory.c
    ${V-REP_ROOT}/V-REP/programming/remoteApi/extApi.h
    ${V-REP_ROOT}/V-REP/programming/remoteApi/extApiPlatform.h
    ${V-REP_ROOT}/V-REP/programming/include/shared_memory.h
    ${V-REP_ROOT}/V-REP/programming/remoteApi/extApiPlatform.c)

  set (EXTRA_LIBRARIES pthread)

  include_directories (${V-REP_PATH}/remoteApi ${V-REP_PATH}/include)
  
  add_definitions( -Dpthread )
  add_definitions (-DNON_MATLAB_PARSING)
  add_definitions (-DMAX_EXT_API_CONNECTIONS=255)
  
  add_library (test_Client SHARED ${SOURCES})
  
  
  add_executable(bubbleRob_test ${SOURCES} )

  target_link_libraries (bubbleRob_test test_Client ${EXTRA_LIBRARIES})
  target_link_libraries (bubbleRob_test test_Client rt)

  
  #Compile the api
  #execute_process (COMMAND "/bin/bash" "-c" "cd ${V-REP_ROOT}/V-REP/programming/v-repSampleApi; qmake bubbleRob_test.pro; make")

else()
  message(STATUS "Kindly Install V-REP")

endif(EXISTS "${PROJECT_ROOT_DIR}/../V-REP")  
