#!/usr/bin/cmake
cmake_minimum_required (VERSION 3.18)

# -------------------------------------------------------------------------------------------------

project(
  WavPack
  VERSION 5.7.0
  DESCRIPTION "Very fast and highly effective data compression format"
)

# This sets a bunch of compile flags and defined ${NUCLEX_COMPILER_TAG} to
# say something like linux-gcc-13.2-amd64-debug. You should have this directory
# if you do a full clone of a project that is using this third-party library build.
include("../../build-system/cmake/cplusplus.cmake")

# -------------------------------------------------------------------------------------------------

# TODO: Effin' CMake again. Can't this abomination of a useless build system just die?
#  If downloads the file. It complains about the checksum if it's wrong.
# Then it writes a BAD status into DOWNLOAD_STATUS. And leaves ERROR_MESSAGE empty.
# Completely useless. Weird, custom build language with side effects as the primary
# means of returning results and that probably changes between versions. Not fixing this.
if(NOT EXISTS ${PROJECT_SOURCE_DIR}/build)
  message(STATUS "Build directory not present, downloading sources")

  file(
    DOWNLOAD https://github.com/dbry/WavPack/archive/refs/tags/5.7.0.tar.gz
    ${PROJECT_SOURCE_DIR}/downloads/5.7.0.tar.gz
    SHOW_PROGRESS
    EXPECTED_HASH SHA256=7875d49596286055b52439ed42f044bd8ad426aa4cc5aabd96bfe7abb971d5e8
  )
  list(GET DOWNLOAD_STATUS 0 STATUS_CODE)
  if(NOT ${STATUS_CODE} EQUAL 0)
    list(GET DOWNLOAD_STATUS 1 ERROR_MESSAGE)
    file(REMOVE ${PROJECT_SOURCE_DIR}/downloads/5.7.0.tar.gz)
    message(FATAL_ERROR "Could not download WavPack: ${ERROR_MESSAGE}")
  endif()

  message(STATUS "Extracting WavPack sources")
  file(
    ARCHIVE_EXTRACT
    INPUT ${PROJECT_SOURCE_DIR}/downloads/5.7.0.tar.gz
    DESTINATION ${CMAKE_BINARY_DIR}/extract
  )
  file(
    RENAME
    ${CMAKE_BINARY_DIR}/extract/5.7.0
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
  sourceFiles
  "build/src/write_words.c"
  "build/src/tags.c"
  "build/src/extra1.c"
  "build/src/open_filename.c"
  "build/src/pack_dsd.c"
  "build/src/unpack_dsd.c"
  "build/src/unpack3_seek.c"
  "build/src/entropy_utils.c"
  "build/src/extra2.c"
  "build/src/unpack_seek.c"
  "build/src/unpack3.c"
  "build/src/pack.c"
  "build/src/unpack_utils.c"
  "build/src/unpack.c"
  "build/src/open_utils.c"
  "build/src/pack_utils.c"
  "build/src/tag_utils.c"
  "build/src/common_utils.c"
  "build/src/unpack_floats.c"
  "build/src/pack_dns.c"
  "build/src/unpack3_open.c"
  "build/src/read_words.c"
  "build/src/pack_floats.c"
  "build/src/decorr_utils.c"
  "build/src/open_legacy.c"
  "build/src/open_raw.c"
)
set(
  headerFiles
  "build/src/wavpack_version.h"
  "build/src/wavpack_local.h"
  "build/src/decorr_tables.h"
  "build/src/unpack3.h"
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
  PUBLIC "build/lib"
)

target_sources(
  WavPack
  PUBLIC ${headerFiles}
  PRIVATE ${sourceFiles}
)

set_target_properties(WavPack PROPERTIES PREFIX "")
set_target_properties(WavPack PROPERTIES OUTPUT_NAME "archive")

# -------------------------------------------------------------------------------------------------

install(
  TARGETS WavPack
  ARCHIVE DESTINATION ${PROJECT_SOURCE_DIR}/bin/${NUCLEX_COMPILER_TAG}
)

# -------------------------------------------------------------------------------------------------

file(
	WRITE "${PROJECT_SOURCE_DIR}/WavPackConfig.cmake"
	"#!/usr/bin/cmake

# Configuration to include WavPack in a CMake-based project. If you want to
# reference WavPack as an externally compiled static library, do this:
#
#    set(WavPack_DIR \"../ThirdParty/wavpack\")
#    find_package(WavPack REQUIRED CONFIG)
#
#    target_link_libraries(
#        MyAwesomeProject
#        PRIVATE WavPack::Static
#    )
#
# Alternatively, if you want to build WavPack together with your project,
# use the normal CMakeLists.txt with CMake's add_subdirectory() command:
#
#    add_subdirectory(
#        \"\${PROJECT_SOURCE_DIR}/../ThirdParty/wavpack\"
#        \"\${CMAKE_BINARY_DIR}/wavpack\"
#    )
#
#    target_link_libraries(
#        MyAwesomeProject
#        PRIVATE WavPack
#    )
#

# -------------------------------------------------------------------------------------------------

if(NOT DEFINED NUCLEX_COMPILER_TAG)
	message(
		FATAL_ERROR
		\"NUCLEX_COMPILER_TAG not defined! Include cplusplus.cmake before importing this package \\
		in order to generate a tag identifying the platform/compiler/architecture/variant!\"
	)
endif()

# -------------------------------------------------------------------------------------------------

if(NOT EXISTS \"\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}\")
	# TODO: Warn and link release build when compiling in debug mode
	# TODO: Warn and link build for older compiler version if found
	message(
		FATAL_ERROR
		\"Directory '\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}' not found. \\
		Please either build and install this project before importing it via \\
		find_package() or use this project's main CMakeFiles.txt via add_subdirectory()!\"
	)
endif()

# -------------------------------------------------------------------------------------------------

add_library(WavPack::Static STATIC IMPORTED)

# This may cause warnings on recent GCC versions (10.0.0+?) with LTO because GCC detects
# that the headers used during build (residing in build/) are not the same used when
# linking the library (copies resising in Include/).
#
# CMake doesn't run the install step during build, so the only way to get the headers
# in place before building would be by copying them rather than installing them.
set_target_properties(
	WavPack::Static PROPERTIES
	INTERFACE_INCLUDE_DIRECTORIES \"\${CMAKE_CURRENT_LIST_DIR}/Include\"
	IMPORTED_LINK_INTERFACE_LANGUAGES \"C\"
)

if(WIN32)
	set_target_properties(
		WavPack::Static PROPERTIES
		IMPORTED_LOCATION \"\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}/wavpack.lib\"
	)
else()
	set_target_properties(
		WavPack::Static PROPERTIES
		IMPORTED_LOCATION \"\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}/wavpack.a\"
	)
endif()

message(STATUS \"Imported WavPack targets with binaries in '\${CMAKE_CURRENT_LIST_DIR}'\")"
)

# -------------------------------------------------------------------------------------------------
