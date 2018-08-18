#   This `CMakeLists.txt' file provides targets representing ICU4C header and
# library files. The ICU4C header and library files will be searched or
# installed, depending on variables set before this file is included.
#
#   If `EXT_ICU4C_DOWNLOAD' is set and evaluates to true, then an ICU4C source
# tree will be downloaded. The version to be downloaded is
#
#   - `EXT_ICU4C_VERSION' if it is specified, otherwise
#   - the latest release version.
#
# The source tree will be located at
#
#   - `EXT_ICU4C_SOURCE_DIR' if it is specified, otherwise
#   - `${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix/src/icu4c'.
#
#   Next, if `EXT_ICU4C_DOWLOAD' or `EXT_ICU4C_SOURCE_DIR' is set and
# evaluates to true, then ICU4C will be built. The following variables affect
# the configuration of the ICU4C libraries to be built.
#
#   - EXT_ICU4C_BUILD_TYPE: Allowed values are `debug' and `release'. If not
#     set, `CMAKE_BUILD_TYPE' determines this value.
#   - EXT_ICU4C_PREFIX: The prefix of the locations of header and library
#     files to be installed. The default value is
#     `${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix'.
#   - EXT_ICU4C_WITH_STD: The C++ language standard. Allowed values are `11',
#     `14', and `17'. If not set, `CMAKE_CXX_STANDARD' determines this value.
#   - EXT_ICU4C_WITH_ASAN: If set and evaluates to true, library files will be
#     compiled with AddressSanitizer instrumentation.
#   - EXT_ICU4C_WITH_UBSAN: If set and evaluates to true, library files will
#     be compiled with UndefinedSanitizer instrumentation.
#   - EXT_ICU4C_WITH_TSAN: If set and evaluates to true, library files will be
#     compiled with ThreadSanitizer instrumentation.
#   - EXT_ICU4C_WITH_LIBSTDCXX_DEBUG_MODE: If set and evaluates to true,
#     library files will be compiled with libstdc++ debug mode.
#
#   Finally, the following targets represent the ICU4C header and library
# files:
#
#   - ext_icu4c_headers
#   - ext_icu4c_data
#   - ext_icu4c_uc
#   - ext_icu4c_i18n
#   - ext_icu4c_io
#   - ext_icu4c_tu
#   - ext_icu4c_test
#
# If `EXT_ICU4C_DOWLOAD' or `EXT_ICU4C_SOURCE_DIR' is set and evaluates to
# true, then the above targets are IMPORTED targets, which will have been
# built as described above. Otherwise, the above targets are interface
# libraries that just represent compiler and linker options necessary to link
# already installed library files. If the already installed header or library
# files are not in a standard location, you can explicitly specify
# `EXT_ICU4C_PREFIX' to tell the compiler and linker the directories where
# they are installed.

cmake_minimum_required(VERSION 3.12.0)
include(ExternalProject)

if(EXT_ICU4C_DOWNLOAD)
  if(NOT EXT_ICU4C_SOURCE_DIR)
    set(EXT_ICU4C_SOURCE_DIR
      "${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix/src/icu4c")
  endif()
  set(EXT_ICU4C_DOWNLOAD_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/download"
    --debug)
  if(EXT_ICU4C_VERSION)
    set(EXT_ICU4C_DOWNLOAD_COMMAND
      "${EXT_ICU4C_DOWNLOAD_COMMAND}"
      "--version=${EXT_ICU4C_VERSION}")
  endif()
  set(EXT_ICU4C_DOWNLOAD_COMMAND
    "${EXT_ICU4C_DOWNLOAD_COMMAND}"
    "--source-dir=${EXT_ICU4C_SOURCE_DIR}"
    "--clobber-source-dir")
else()
  set(EXT_ICU4C_DOWNLOAD_COMMAND true)
endif()

if(EXT_ICU4C_SOURCE_DIR)
  if(NOT EXT_ICU4C_PREFIX)
    set(EXT_ICU4C_PREFIX
      "${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix")
  endif()
  set(EXT_ICU4C_CONFIGURE_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/configure"
    --debug
    "--source-dir=${EXT_ICU4C_SOURCE_DIR}"
    "--build-dir=<BINARY_DIR>")
  if(NOT EXT_ICU4C_BUILD_TYPE)
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
      set(EXT_ICU4C_BUILD_TYPE "debug")
    elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
      set(EXT_ICU4C_BUILD_TYPE "release")
    else()
      message(FATAL_ERROR "An unsupported build type: CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}")
    endif()
  endif()
  if(EXT_ICU4C_BUILD_TYPE STREQUAL "debug")
    set(EXT_ICU4C_CONFIGURE_COMMAND
      "${EXT_ICU4C_CONFIGURE_COMMAND}"
      "--build-type=debug")
  elseif(EXT_ICU4C_BUILD_TYPE STREQUAL "release")
    set(EXT_ICU4C_CONFIGURE_COMMAND
      "${EXT_ICU4C_CONFIGURE_COMMAND}"
      "--build-type=release")
  else()
    message(
      FATAL_ERROR
      "An invalid value `${EXT_ICU4C_BUILD_TYPE}' for `EXT_ICU4C_BUILD_TYPE' variable.")
  endif()
  if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
      set(EXT_ICU4C_CONFIGURE_COMMAND
        "${EXT_ICU4C_CONFIGURE_COMMAND}"
        "--platform=Linux")
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
      set(EXT_ICU4C_CONFIGURE_COMMAND
        "${EXT_ICU4C_CONFIGURE_COMMAND}"
        "--platform=Linux/gcc")
    else()
      message(FATAL_ERROR
        "An unsupported compiler: CMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID}.")
    endif()
  else()
    message(FATAL_ERROR
      "An unsupported system: CMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}.")
  endif()
  set(EXT_ICU4C_CONFIGURE_COMMAND
    "${EXT_ICU4C_CONFIGURE_COMMAND}"
    "--prefix=${EXT_ICU4C_PREFIX}")
  if(NOT EXT_ICU4C_WITH_STD)
    if(CMAKE_CXX_STANDARD STREQUAL "11")
      set(EXT_ICU4C_WITH_STD "11")
    elseif(CMAKE_CXX_STANDARD STREQUAL "14")
      set(EXT_ICU4C_WITH_STD "14")
    elseif(CMAKE_CXX_STANDARD STREQUAL "17")
      set(EXT_ICU4C_WITH_STD "17")
    else()
      message(FATAL_ERROR "An unsupported c++ language standard: CMAKE_CXX_STANDARD=${CMAKE_CXX_STANDARD}")
    endif()
  endif()
  if(EXT_ICU4C_WITH_STD STREQUAL "11")
    set(EXT_ICU4C_CONFIGURE_COMMAND
      "${EXT_ICU4C_CONFIGURE_COMMAND}"
      "--with-std=11")
  elseif(EXT_ICU4C_WITH_STD STREQUAL "14")
    set(EXT_ICU4C_CONFIGURE_COMMAND
      "${EXT_ICU4C_CONFIGURE_COMMAND}"
      "--with-std=14")
  elseif(EXT_ICU4C_WITH_STD STREQUAL "17")
    set(EXT_ICU4C_CONFIGURE_COMMAND
      "${EXT_ICU4C_CONFIGURE_COMMAND}"
      "--with-std=17")
  else()
    message(
      FATAL_ERROR
      "An invalid value `${EXT_ICU4C_WITH_STD}' for `EXT_ICU4C_WITH_STD' variable.")
  endif()
  if(EXT_ICU4C_WITH_ASAN)
    set(EXT_ICU4C_CONFIGURE_COMMAND
      "${EXT_ICU4C_CONFIGURE_COMMAND}"
      "--with-asan")
  endif()
  if(EXT_ICU4C_WITH_UBSAN)
    set(EXT_ICU4C_CONFIGURE_COMMAND
      "${EXT_ICU4C_CONFIGURE_COMMAND}"
      "--with-ubsan")
  endif()
  if(EXT_ICU4C_WITH_TSAN)
    set(EXT_ICU4C_CONFIGURE_COMMAND
      "${EXT_ICU4C_CONFIGURE_COMMAND}"
      "--with-tsan")
  endif()
  if(EXT_ICU4C_WITH_LIBSTDCXX_DEBUG_MODE)
    set(EXT_ICU4C_CONFIGURE_COMMAND
      "${EXT_ICU4C_CONFIGURE_COMMAND}"
      "--with-libstdc++-debug-mode")
  endif()

  set(EXT_ICU4C_BUILD_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/build"
    --debug
    "--build-dir=<BINARY_DIR>")

  set(EXT_ICU4C_INSTALL_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/install"
    "--build-dir=<BINARY_DIR>")

  ExternalProject_Add(icu4c
    DOWNLOAD_COMMAND ${EXT_ICU4C_DOWNLOAD_COMMAND}
    CONFIGURE_COMMAND ${EXT_ICU4C_CONFIGURE_COMMAND}
    BUILD_COMMAND ${EXT_ICU4C_BUILD_COMMAND}
    INSTALL_COMMAND ${EXT_ICU4C_INSTALL_COMMAND}
    TEST_COMMAND "")

  add_library(ext_icu4c_headers
    INTERFACE)
  target_include_directories(ext_icu4c_headers
    INTERFACE
    "${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix/include")
  add_dependencies(ext_icu4c_headers
    icu4c)

  if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    add_library(ext_icu4c_data
      SHARED
      IMPORTED
      GLOBAL)
    set_target_properties(ext_icu4c_data
      PROPERTIES
      IMPORTED_LOCATION
      "${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix/lib/libicudata.so")
    target_link_libraries(ext_icu4c_data
      INTERFACE
      ext_icu4c_headers)
    add_dependencies(ext_icu4c_data
      icu4c)

    add_library(ext_icu4c_uc
      SHARED
      IMPORTED
      GLOBAL)
    set_target_properties(ext_icu4c_uc
      PROPERTIES
      IMPORTED_LOCATION
      "${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix/lib/libicuuc.so")
    target_link_libraries(ext_icu4c_uc
      INTERFACE
      ext_icu4c_headers
      ext_icu4c_data)
    add_dependencies(ext_icu4c_uc
      icu4c)

    add_library(ext_icu4c_i18n
      SHARED
      IMPORTED
      GLOBAL)
    set_target_properties(ext_icu4c_i18n
      PROPERTIES
      IMPORTED_LOCATION
      "${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix/lib/libicui18n.so")
    target_link_libraries(ext_icu4c_i18n
      INTERFACE
      ext_icu4c_headers
      ext_icu4c_data
      ext_icu4c_uc)
    add_dependencies(ext_icu4c_i18n
      icu4c)

    add_library(ext_icu4c_io
      SHARED
      IMPORTED
      GLOBAL)
    set_target_properties(ext_icu4c_io
      PROPERTIES
      IMPORTED_LOCATION
      "${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix/lib/libicuio.so")
    target_link_libraries(ext_icu4c_io
      INTERFACE
      ext_icu4c_headers
      ext_icu4c_data
      ext_icu4c_uc
      ext_icu4c_i18n)
    add_dependencies(ext_icu4c_io
      icu4c)

    add_library(ext_icu4c_tu
      SHARED
      IMPORTED
      GLOBAL)
    set_target_properties(ext_icu4c_tu
      PROPERTIES
      IMPORTED_LOCATION
      "${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix/lib/libicutu.so")
    target_link_libraries(ext_icu4c_tu
      INTERFACE
      ext_icu4c_headers
      ext_icu4c_data
      ext_icu4c_uc
      ext_icu4c_i18n)
    add_dependencies(ext_icu4c_tu
      icu4c)

    add_library(ext_icu4c_test
      SHARED
      IMPORTED
      GLOBAL)
    set_target_properties(ext_icu4c_test
      PROPERTIES
      IMPORTED_LOCATION
      "${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix/lib/libicutest.so")
    target_link_libraries(ext_icu4c_test
      INTERFACE
      ext_icu4c_headers
      ext_icu4c_data
      ext_icu4c_uc
      ext_icu4c_i18n
      ext_icu4c_tu)
    add_dependencies(ext_icu4c_test
      icu4c)
  else()
    message(FATAL_ERROR "An unsupported system: CMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}.")
  endif()
else()
  add_library(ext_icu4c_headers
    INTERFACE)
  if(EXT_ICU4C_PREFIX)
    set_target_include_directories(ext_icu4c_headers
      INTERFACE
      "${EXT_ICU4C_PREFIX}/include")
  endif()

  if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang"
      OR CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    if(EXT_ICU4C_PREFIX)
      set(EXT_ICU4C_LINKER_SEARCH_FLAGS
        "-L${EXT_ICU4C_PREFIX}/lib")
    endif()

    add_library(ext_icu4c_data
      INTERFACE)
    target_link_libraries(ext_icu4c_data
      ext_icu4c_headers
      ${EXT_ICU4C_LINKER_SEARCH_FLAGS}
      "-licudata")

    add_library(ext_icu4c_uc
      INTERFACE)
    target_link_libraries(ext_icu4c_uc
      ext_icu4c_headers
      ext_icu4c_data
      ${EXT_ICU4C_LINKER_SEARCH_FLAGS}
      "-licuuc")

    add_library(ext_icu4c_i18n
      INTERFACE)
    target_link_libraries(ext_icu4c_i18n
      ext_icu4c_headers
      ext_icu4c_data
      ext_icu4c_uc
      ${EXT_ICU4C_LINKER_SEARCH_FLAGS}
      "-licui18n")

    add_library(ext_icu4c_io
      INTERFACE)
    target_link_libraries(ext_icu4c_io
      ext_icu4c_headers
      ext_icu4c_data
      ext_icu4c_uc
      ext_icu4c_i18n
      ${EXT_ICU4C_LINKER_SEARCH_FLAGS}
      "-licuio")

    add_library(ext_icu4c_tu
      INTERFACE)
    target_link_libraries(ext_icu4c_tu
      ext_icu4c_headers
      ext_icu4c_data
      ext_icu4c_uc
      ext_icu4c_i18n
      ${EXT_ICU4C_LINKER_SEARCH_FLAGS}
      "-licutu")

    add_library(ext_icu4c_test
      INTERFACE)
    target_link_libraries(ext_icu4c_test
      ext_icu4c_headers
      ext_icu4c_data
      ext_icu4c_uc
      ext_icu4c_i18n
      ext_icu4c_tu
      ${EXT_ICU4C_LINKER_SEARCH_FLAGS}
      "-licutest")
  else()
    message(FATAL_ERROR
      "An unsupported compiler: CMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID}.")
  endif()
endif()
