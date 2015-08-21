cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME flatbuffers)
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
  set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
  include(CPM)
else()
  set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
  find_package(Git)
  if(NOT GIT_FOUND)
    message(FATAL_ERROR "CPM requires Git.")
  endif()
  if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
    execute_process(
      COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
      RESULT_VARIABLE error_code
      OUTPUT_VARIABLE head_sha)
    if(error_code)
      message(FATAL_ERROR "CPM failed to get the hash for HEAD")
    endif()
  endif()
  include(${CPM_DIR}/CPM.cmake)
endif()

# All externals *must* define this.
CPM_ForceOnlyOneModuleVersion()

CPM_InitModule(${CPM_MODULE_NAME})

#------------------------------------------------------------------------------
# Flatbuffers
#------------------------------------------------------------------------------
# flatbuffers as a subdirectory so that it can capture your compiler flags
set(FLATBUFFERS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/3rdParty/flatbuffers)
CPM_EnsureRepoIsCurrent(
  TARGET_DIR ${FLATBUFFERS_DIR}
  GIT_REPOSITORY "https://github.com/google/flatbuffers/"
  GIT_TAG origin/master
  USE_CACHING TRUE
  )

set(FlatBuffers_Library_SRCS
  ${FLATBUFFERS_DIR}/include/flatbuffers/flatbuffers.h
  ${FLATBUFFERS_DIR}/include/flatbuffers/hash.h
  ${FLATBUFFERS_DIR}/include/flatbuffers/idl.h
  ${FLATBUFFERS_DIR}/include/flatbuffers/util.h
  ${FLATBUFFERS_DIR}/include/flatbuffers/reflection.h
  ${FLATBUFFERS_DIR}/include/flatbuffers/reflection_generated.h
  ${FLATBUFFERS_DIR}/src/idl_parser.cpp
  ${FLATBUFFERS_DIR}/src/idl_gen_text.cpp
  ${FLATBUFFERS_DIR}/src/reflection.cpp
)

set(FlatBuffers_Compiler_SRCS
  ${FlatBuffers_Library_SRCS}
  ${FLATBUFFERS_DIR}/src/idl_gen_cpp.cpp
  ${FLATBUFFERS_DIR}/src/idl_gen_general.cpp
  ${FLATBUFFERS_DIR}/src/idl_gen_go.cpp
  ${FLATBUFFERS_DIR}/src/idl_gen_python.cpp
  ${FLATBUFFERS_DIR}/src/idl_gen_fbs.cpp
  ${FLATBUFFERS_DIR}/src/flatc.cpp
)

if(APPLE)
  set(CMAKE_CXX_FLAGS
    "${CMAKE_CXX_FLAGS} -std=c++11 -stdlib=libc++ -Wall -pedantic -Werror -Wextra")
elseif(CMAKE_COMPILER_IS_GNUCXX)
  set(CMAKE_CXX_FLAGS
    "${CMAKE_CXX_FLAGS} -std=c++0x -Wall -pedantic -Werror -Wextra")
elseif("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
  set(CMAKE_CXX_FLAGS
    "${CMAKE_CXX_FLAGS} -std=c++0x -stdlib=libc++ -Wall -pedantic -Werror -Wextra")
endif()

#-----------------------------------------------------------------------
# Library setup
#-----------------------------------------------------------------------

# Build the library.
include_directories("${FLATBUFFERS_DIR}/include")
add_library(${CPM_MODULE_NAME} STATIC ${FlatBuffers_Library_SRCS})
CPM_ExportAdditionalIncludeDir("${FLATBUFFERS_DIR}/include")
CPM_ExportAdditionalLibraryTarget(${CPM_MODULE_NAME})

# Compile flatc
add_executable(flatc ${FlatBuffers_Compiler_SRCS})

function(compile_flatbuffers_schema_to_cpp SRC_FBS TARGET_NAME)
  get_filename_component(SRC_FBS_DIR ${SRC_FBS} PATH)
  string(REGEX REPLACE "\\.fbs$" "_generated.h" GEN_HEADER ${SRC_FBS})
  add_custom_command(
    OUTPUT ${GEN_HEADER}
    COMMAND flatc -c --no-includes --gen-mutable -o "${SRC_FBS_DIR}" "${CMAKE_CURRENT_SOURCE_DIR}/${SRC_FBS}"
    DEPENDS flatc)
  add_custom_target(${TARGET_NAME} DEPENDS ${GEN_HEADER})
endfunction()


