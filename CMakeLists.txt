#!/usr/bin/cmake
cmake_minimum_required (VERSION 3.18)

# -------------------------------------------------------------------------------------------------

project(
  WavPack
  VERSION 5.7.0
  DESCRIPTION "Lossless audio compression format that can handle floating point channels"
)

# This sets a bunch of compile flags and defined ${NUCLEX_COMPILER_TAG} to
# say something like linux-gcc-13.2-amd64-debug. You should have this directory
# if you do a full clone of a project that is using this third-party library build.
include("../../build-system/cmake/cplusplus.cmake")

# -------------------------------------------------------------------------------------------------

if(NOT EXISTS ${PROJECT_SOURCE_DIR}/build)
  message(STATUS "Build directory not present, downloading sources")

  file(
    DOWNLOAD https://github.com/dbry/WavPack/archive/5.7.0.tar.gz
    ${PROJECT_SOURCE_DIR}/downloads/5.7.0.tar.gz
    SHOW_PROGRESS
    EXPECTED_HASH SHA256=c5742ba1054d36ff3d22f0101a9be066f55f6becb9b2a7352c79fa362f2d3d76
    STATUS DOWNLOAD_STATUS
  )
  list(GET DOWNLOAD_STATUS 0 STATUS_CODE)
  if(NOT ${STATUS_CODE} EQUAL 0)
    list(GET DOWNLOAD_STATUS 1 ERROR_MESSAGE)
    list(GET DOWNLOAD_STATUS 2 HTTP_CODE)
    message(
      FATAL_ERROR
      "Could not download WavPack sources: ${ERROR_MESSAGE} (HTTP result ${HTTP_CODE})"
    )
  endif()

  message(STATUS "Extracting WavPack sources")
  file(
    ARCHIVE_EXTRACT
    INPUT ${PROJECT_SOURCE_DIR}/downloads/5.7.0.tar.gz
    DESTINATION ${CMAKE_BINARY_DIR}/extract
  )
  file(
    RENAME
    ${CMAKE_BINARY_DIR}/extract/WavPack-5.7.0
    ${PROJECT_SOURCE_DIR}/build
  )
endif()

# -------------------------------------------------------------------------------------------------

#configure_file(
#	"${PROJECT_SOURCE_DIR}/config.h"
#	"${PROJECT_SOURCE_DIR}/build/config.h"
#	COPYONLY
#)

# -------------------------------------------------------------------------------------------------

set(
  headerFiles
  "build/include/wavpack.h"
  #"build/src/wavpack_version.h"
  #"build/src/wavpack_local.h"
  #"build/src/decorr_tables.h"
  #"build/src/unpack3.h"
)
set(
  sourceFiles
  "build/src/common_utils.c"
  "build/src/decorr_utils.c"
  "build/src/entropy_utils.c"
  "build/src/extra1.c"
  "build/src/extra2.c"
  "build/src/open_filename.c"
  "build/src/open_legacy.c"
  "build/src/open_raw.c"
  "build/src/open_utils.c"
  "build/src/pack.c"
  "build/src/pack_dns.c"
  "build/src/pack_dsd.c"
  "build/src/pack_floats.c"
  "build/src/pack_utils.c"
  "build/src/read_words.c"
  "build/src/tag_utils.c"
  "build/src/tags.c"
  "build/src/unpack.c"
  "build/src/unpack_dsd.c"
  "build/src/unpack_floats.c"
  "build/src/unpack_seek.c"
  "build/src/unpack_utils.c"
  "build/src/unpack3_open.c"
  "build/src/unpack3_seek.c"
  "build/src/unpack3.c"
  "build/src/write_words.c"
)

# -------------------------------------------------------------------------------------------------

add_library(WavPack STATIC)
add_library(WavPack::Static ALIAS WavPack)

target_compile_definitions(
  WavPack
  PRIVATE WAVPACK_EXPORTS
)

target_include_directories(
  WavPack
  PUBLIC "build/include"
)

target_sources(
  WavPack
  PUBLIC ${headerFiles}
  PRIVATE ${sourceFiles}
)

set_target_properties(WavPack PROPERTIES PREFIX "")
set_target_properties(WavPack PROPERTIES OUTPUT_NAME "wavpack")

# -------------------------------------------------------------------------------------------------

install(
  TARGETS WavPack
  ARCHIVE DESTINATION ${PROJECT_SOURCE_DIR}/bin/${NUCLEX_COMPILER_TAG}
)
