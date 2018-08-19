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
#   - EXT_ICU4C_BUILD_TYPE: Specify the build type of library files. Allowed
#     values are `debug' and `release'. The default value is determined based
#     on the value of `CMAKE_BUILD_TYPE' variable.
#   - EXT_ICU4C_PREFIX: The prefix of the locations of header and library
#     files to install. The default value is
#     `${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix'.
#   - EXT_ICU4C_SHARED_LIBS: If this variable is set and evaluates to true,
#     shared library files will be built. Otherwise, static library files
#     will. The default value is determined based on the value of
#     `BUILD_SHARED_LIBS' variable.
#   - EXT_ICU4C_CXX_STANDARD: Compile library files with the specified C++
#     language standard. Allowed values are `11', `14', and `17'. The default
#     value is determined based on the value of `CMAKE_CXX_STANDARD' value.
#   - EXT_ICU4C_WITH_ASAN: If this variable is set and evaluates to true,
#     library files will compile and link with AddressSanitizer
#     instrumentation.
#   - EXT_ICU4C_WITH_UBSAN: If this variable is set and evaluates to true,
#     library files will compile and link with UndefinedSanitizer
#     instrumentation.
#   - EXT_ICU4C_WITH_TSAN: If this variable is set and evaluates to true,
#     library files will compile and link with ThreadSanitizer
#     instrumentation.
#   - EXT_ICU4C_WITH_LIBSTDCXX_DEBUG_MODE: If this variable is set and
#     evaluates to true, library files will compile with libstdc++ debug mode.
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

include(ExternalProject)

if(EXT_ICU4C_DOWNLOAD)
  set(EXT_ICU4C_DOWNLOAD_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/download")
  if(EXT_ICU4C_VERSION)
    list(APPEND
      EXT_ICU4C_DOWNLOAD_COMMAND
      "--version=${EXT_ICU4C_VERSION}")
  endif()
  if(NOT EXT_ICU4C_SOURCE_DIR)
    set(EXT_ICU4C_SOURCE_DIR
      "${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix/src/icu4c"
      CACHE
      PATH
      "ICU4C source directory.")
  endif()
  list(APPEND
    EXT_ICU4C_DOWNLOAD_COMMAND
    "--source-dir=${EXT_ICU4C_SOURCE_DIR}"
    "--clobber-source-dir")
else()
  set(EXT_ICU4C_DOWNLOAD_COMMAND
    "true")
endif()

if(EXT_ICU4C_SOURCE_DIR)
  set(EXT_ICU4C_CONFIGURE_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/configure"
    "--source-dir=${EXT_ICU4C_SOURCE_DIR}"
    "--build-dir=<BINARY_DIR>")
  if(NOT EXT_ICU4C_BUILD_TYPE)
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
      set(EXT_ICU4C_BUILD_TYPE
        "debug"
        CACHE
        STRING
        "The build type of ICU4C (allowed values are `debug' and `release').")
    elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
      set(EXT_ICU4C_BUILD_TYPE
        "release"
        CACHE
        STRING
        "The build type of ICU4C (allowed values are `debug' and `release').")
    else()
      message(FATAL_ERROR
        "An unsupported build type: CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}")
    endif()
  endif()
  if(EXT_ICU4C_BUILD_TYPE STREQUAL "debug")
    list(APPEND
      EXT_ICU4C_CONFIGURE_COMMAND
      "--build-type=debug")
  elseif(EXT_ICU4C_BUILD_TYPE STREQUAL "release")
    list(APPEND
      EXT_ICU4C_CONFIGURE_COMMAND
      "--build-type=release")
  else()
    message(FATAL_ERROR
      "An invalid value `${EXT_ICU4C_BUILD_TYPE}' for `EXT_ICU4C_BUILD_TYPE' variable.")
  endif()
  if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
      list(APPEND
        EXT_ICU4C_CONFIGURE_COMMAND
        "--platform=Linux")
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
      list(APPEND
        EXT_ICU4C_CONFIGURE_COMMAND
        "--platform=Linux/gcc")
    else()
      message(FATAL_ERROR
        "An unsupported compiler: CMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID}.")
    endif()
  else()
    message(FATAL_ERROR
      "An unsupported system: CMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}.")
  endif()
  if(NOT EXT_ICU4C_PREFIX)
    set(EXT_ICU4C_PREFIX
      "${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix"
      CACHE
      PATH
      "The prefix")
  endif()
  list(APPEND
    EXT_ICU4C_CONFIGURE_COMMAND
    "--prefix=${EXT_ICU4C_PREFIX}")
  if(NOT EXT_ICU4C_SHARED_LIBS AND BUILD_SHARED_LIBS)
    set(EXT_ICU4C_SHARED_LIBS
      "ON")
  endif()
  if(NOT BUILD_SHARED_LIBS)
    list(APPEND
      EXT_ICU4C_CONFIGURE_COMMAND
      "--disable-shared"
      "--enable-static")
  endif()
  if(NOT EXT_ICU4C_CXX_STANDARD)
    if(CMAKE_CXX_STANDARD STREQUAL "11")
      set(EXT_ICU4C_CXX_STANDARD
        "11"
        CACHE
        STRING
        "The C++ language standard for ICU4C libraries.")
    elseif(CMAKE_CXX_STANDARD STREQUAL "14")
      set(EXT_ICU4C_CXX_STANDARD
        "14"
        CACHE
        STRING
        "The C++ language standard for ICU4C libraries.")
    elseif(CMAKE_CXX_STANDARD STREQUAL "17")
      set(EXT_ICU4C_CXX_STANDARD
        "17"
        CACHE
        STRING
        "The C++ language standard for ICU4C libraries.")
    else()
      message(FATAL_ERROR
        "An unsupported c++ language standard: CMAKE_CXX_STANDARD=${CMAKE_CXX_STANDARD}")
    endif()
  endif()
  if(EXT_ICU4C_CXX_STANDARD STREQUAL "11")
    list(APPEND
      EXT_ICU4C_CONFIGURE_COMMAND
      "--c++-standard=11")
  elseif(EXT_ICU4C_CXX_STANDARD STREQUAL "14")
    list(APPEND
      EXT_ICU4C_CONFIGURE_COMMAND
      "--c++-standard=14")
  elseif(EXT_ICU4C_CXX_STANDARD STREQUAL "17")
    list(APPEND
      EXT_ICU4C_CONFIGURE_COMMAND
      "--c++-standard=17")
  else()
    message(FATAL_ERROR
      "An invalid value `${EXT_ICU4C_CXX_STANDARD}' for `EXT_ICU4C_CXX_STANDARD' variable.")
  endif()
  if(EXT_ICU4C_WITH_ASAN)
    list(APPEND
      EXT_ICU4C_CONFIGURE_COMMAND
      "--with-asan")
  endif()
  if(EXT_ICU4C_WITH_UBSAN)
    list(APPEND
      EXT_ICU4C_CONFIGURE_COMMAND
      "--with-ubsan")
  endif()
  if(EXT_ICU4C_WITH_TSAN)
    list(APPEND
      EXT_ICU4C_CONFIGURE_COMMAND
      "--with-tsan")
  endif()
  if(EXT_ICU4C_WITH_LIBSTDCXX_DEBUG_MODE)
    list(APPEND
      EXT_ICU4C_CONFIGURE_COMMAND
      "--with-libstdc++-debug-mode")
  endif()

  set(EXT_ICU4C_BUILD_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/build"
    "--build-dir=<BINARY_DIR>")

  set(EXT_ICU4C_INSTALL_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/install"
    "--build-dir=<BINARY_DIR>")

  ExternalProject_Add(ext_icu4c
    PREFIX "icu4c-prefix"
    DOWNLOAD_COMMAND ${EXT_ICU4C_DOWNLOAD_COMMAND}
    CONFIGURE_COMMAND ${EXT_ICU4C_CONFIGURE_COMMAND}
    BUILD_COMMAND ${EXT_ICU4C_BUILD_COMMAND}
    INSTALL_COMMAND ${EXT_ICU4C_INSTALL_COMMAND}
    TEST_COMMAND "")

  add_library(ext_icu4c_headers
    INTERFACE)
  target_include_directories(ext_icu4c_headers
    INTERFACE
    "${EXT_ICU4C_PREFIX}/include")
  add_dependencies(ext_icu4c_headers
    ext_icu4c)

  if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    if(NOT EXT_ICU4C_SHARED_LIBS)
      set(EXT_ICU4C_LIBRARY_TYPE
        "STATIC")
      set(EXT_ICU4C_LIBRARY_EXTENSION
        ".a")
    else()
      set(EXT_ICU4C_LIBRARY_TYPE
        "SHARED")
      set(EXT_ICU4C_LIBRARY_EXTENSION
        ".so")
    endif()
    foreach(spec "data" "uc;data" "i18n;data;uc" "io;data;uc;i18n"
        "tu;data;uc;i18n" "test;data;uc;i18n;tu")
      list(GET spec 0 lib)
      add_library("ext_icu4c_${lib}"
        "${EXT_ICU4C_LIBRARY_TYPE}"
        IMPORTED
        GLOBAL)
      set_target_properties("ext_icu4c_${lib}"
        PROPERTIES
        IMPORTED_LOCATION
        "${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix/lib/lib${lib}${EXT_ICU4C_LIBRARY_EXTENSION}")
      target_link_libraries("ext_icu4c_${lib}"
        INTERFACE
        ext_icu4c_headers)
      list(LENGTH spec len)
      if(len GREATER_EQUAL 2)
        list(REMOVE_AT spec 0)
        foreach(dep IN LISTS spec)
          target_link_libraries("ext_icu4c_${lib}"
            INTERFACE
            "ext_icu4c_${dep}")
        endforeach()
      endif()
      add_dependencies("ext_icu4c_${lib}"
        ext_icu4c)
    endforeach()
  else()
    message(FATAL_ERROR
      "An unsupported system: CMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}.")
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
    add_library(ext_icu4c_link_search_flags
      INTERFACE)
    if(EXT_ICU4C_PREFIX)
      target_link_libraries(ext_icu4c_link_search_flags
        INTERFACE
        "-L${EXT_ICU4C_PREFIX}/lib")
    endif()

    foreach(spec "data" "uc;data" "i18n;data;uc" "io;data;uc;i18n"
        "tu;data;uc;i18n;" "test;data;uc;i18n;tu")
      list(GET spec 0 lib)
      add_library("ext_icu4c_${lib}"
        INTERFACE)
      target_link_libraries("ext_icu4c_${lib}"
        ext_icu4c_headers
        ext_icu4c_link_search_flags)
      list(LENGTH spec len)
      if(len GREATER_EQUAL 2)
        list(REMOVE_AT spec 0)
        foreach(dep IN LISTS spec)
          target_link_libraries("ext_icu4c_${lib}"
            "ext_icu4c_${dep}")
        endforeach()
      endif()
      target_link_libraries("ext_icu4c_${lib}"
        "-licu${lib}")
    endforeach()
  else()
    message(FATAL_ERROR
      "An unsupported compiler: CMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID}.")
  endif()
endif()
