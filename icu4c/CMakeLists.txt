#   This `CMakeLists.txt' file provides targets representing ICU4C library
# files. The ICU4C library files will be searched or installed, depending on
# variables set before this file is included.
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
#   - EXT_ICU4C_PREFIX: The location where architecture-independent files are
#     installed. The default value is
#     `${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix'.
#   - EXT_ICU4C_BUILD_TYPE: Specify the build type of library files. Allowed
#     values are `debug' and `release'. If this variable is not defined, then
#     then `${CMAKE_BUILD_TYPE}' is used to determine the default value.
#   - EXT_ICU4C_USE_STATIC_LIBRARY: If this variable is defined and evaluates
#     to true, build the static library files. Otherwise, build shared ones.
#   - EXT_ICU4C_CXX_STANDARD: Compile library files with the specified C++
#     language standard. Allowed values are `11', `14', and `17'. If this
#     variable is not defined, then `${CMAKE_CXX_STANDARD}' is used as the
#     default value.
#   - EXT_ICU4C_WITH_ASAN: If this variable is set and evaluates to true,
#     library files will compile and link with AddressSanitizer
#     instrumentation.
#   - EXT_ICU4C_WITH_PCSAN: If this variable is set and evaluates to true,
#     library files will compile and link with instrumentation to check
#     pointer comparisons.
#   - EXT_ICU4C_WITH_PSSAN: If this variable is set and evaluates to true,
#     library files will compile and link with instrumentation to check
#     pointer subtractions.
#   - EXT_ICU4C_WITH_TSAN: If this variable is set and evaluates to true,
#     library files will compile and link with ThreadSanitizer
#     instrumentation.
#   - EXT_ICU4C_WITH_LSAN: If this variable is set and evaluates to true,
#     library files will compile and link with LeakSanitizer instrumentation.
#   - EXT_ICU4C_WITH_UBSAN: If this variable is set and evaluates to true,
#     library files will compile and link with UndefinedBehaviorSanitizer
#     instrumentation.
#   - EXT_ICU4C_WITH_LIBSTDCXX_DEBUG_MODE: If this variable is set and
#     evaluates to true, library files will compile with libstdc++ debug mode.
#
#   Finally, the following targets represent the ICU4C library files:
#
#   - ext_icu4c_data
#   - ext_icu4c_uc
#   - ext_icu4c_i18n
#   - ext_icu4c_io
#   - ext_icu4c_tu
#   - ext_icu4c_test
#
# If `EXT_ICU4C_DOWLOAD' or `EXT_ICU4C_SOURCE_DIR' is set and evaluates to
# true, then the above targets are INTERFACE libraries that represent compiler
# and linker options necessary to link the library files to be built.
# Otherwise, the above targets are INTERFACE libraries that represent compiler
# and linker options necessary to search and link already installed library
# files. If the already installed header or library files are not in a
# standard location, you can explicitly define `EXT_ICU4C_PREFIX' variable to
# tell the compiler and linker the directories where they are located.

cmake_minimum_required(VERSION 3.12.0)
include(ExternalProject)

if((DEFINED EXT_ICU4C_SOURCE_DIR)
    AND (NOT IS_ABSOLUTE "${EXT_ICU4C_SOURCE_DIR}"))
  message(FATAL_ERROR
    "A relative path `${EXT_ICU4C_SOURCE_DIR}' is specified for "
    "`EXT_ICU4C_SOURCE_DIR' variable, but is expected to be an absolute one.")
endif()

if("${EXT_ICU4C_DOWNLOAD}")
  set(EXT_ICU4C_DOWNLOAD_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/download")
  if(DEFINED EXT_ICU4C_VERSION)
    list(APPEND
      EXT_ICU4C_DOWNLOAD_COMMAND
      "--version=${EXT_ICU4C_VERSION}")
  endif()
  if(NOT DEFINED EXT_ICU4C_SOURCE_DIR)
    set(EXT_ICU4C_SOURCE_DIR
      "${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix/src/icu4c")
  endif()
  list(APPEND
    EXT_ICU4C_DOWNLOAD_COMMAND
    "--source-dir=${EXT_ICU4C_SOURCE_DIR}"
    --clobber-source-dir)
else()
  set(EXT_ICU4C_DOWNLOAD_COMMAND
    true)
endif()

if(DEFINED EXT_ICU4C_SOURCE_DIR)
  set(EXT_ICU4C_CONFIGURE_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/configure")

  if(NOT DEFINED EXT_ICU4C_BUILD_TYPE)
    if("${EXT_BUILD_TYPE}" STREQUAL "debug")
      set(EXT_ICU4C_BUILD_TYPE
        debug)
    elseif("${EXT_BUILD_TYPE}" STREQUAL "release")
      set(EXT_ICU4C_BUILD_TYPE
        release)
    else()
      message(FATAL_ERROR
        "An unsupported build type: CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}.")
    endif()
  endif()

  if(EXT_ICU4C_BUILD_TYPE STREQUAL "debug")
    list(APPEND
      EXT_ICU4C_CONFIGURE_COMMAND
      --enable-debug
      --disable-release)
  elseif(EXT_ICU4C_BUILD_TYPE STREQUAL "release")
    # Do nothing.
  else()
    message(FATAL_ERROR
      "An invalid value `${EXT_ICU4C_BUILD_TYPE}' for `EXT_ICU4C_BUILD_TYPE' "
      "variable.")
  endif()

  if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
      list(APPEND
        EXT_ICU4C_CONFIGURE_COMMAND
        --platform=Linux)
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
      # Do not use `Linux/gcc` because it assumes `CC=gcc` and `CXX=g++`.
      # We want `CC` to be set to the value of `${CMAKE_C_COMPILER}` and `CXX`
      # to be set to the value of `${CMAKE_CXX_COMPILER}`.
      list(APPEND
        EXT_ICU4C_CONFIGURE_COMMAND
        --platform=Linux)
    else()
      message(FATAL_ERROR
        "An unsupported C++ compiler: "
        "CMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID}.")
    endif()
  else()
    message(FATAL_ERROR
      "An unsupported system: CMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}.")
  endif()

  if(NOT DEFINED EXT_ICU4C_CXX_STANDARD)
    if("${EXT_CXX_STANDARD}" STREQUAL "")
      # Do nothing.
    elseif("${EXT_CXX_STANDARD}" STREQUAL "11")
      set(EXT_ICU4C_CXX_STANDARD
        11)
    elseif("${EXT_CXX_STANDARD}" STREQUAL "14")
      set(EXT_ICU4C_CXX_STANDARD
        14)
    elseif("${EXT_CXX_STANDARD}" STREQUAL "17")
      set(EXT_ICU4C_CXX_STANDARD
        17)
    else()
      message(FATAL_ERROR
        "An unsupported C++ language standard: "
        "CMAKE_CXX_STANDARD=${CMAKE_CXX_STANDARD}")
    endif()
  endif()

  if((CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
      OR (CMAKE_CXX_COMPILER_ID STREQUAL "GNU"))
    if("${EXT_ICU4C_CXX_STANDARD}" STREQUAL "")
      # Do nothing.
    elseif("${EXT_ICU4C_CXX_STANDARD}" STREQUAL "11")
      list(APPEND
        EXT_ICU4C_CXXFLAGS
        -std=c++11)
    elseif("${EXT_ICU4C_CXX_STANDARD}" STREQUAL "14")
      list(APPEND
        EXT_ICU4C_CXXFLAGS
        -std=c++14)
    elseif("${EXT_ICU4C_CXX_STANDARD}" STREQUAL "17")
      list(APPEND
        EXT_ICU4C_CXXFLAGS
        -std=c++17)
    else()
      message(FATAL_ERROR
        "An invalid value `${EXT_ICU4C_CXX_STANDARD}' for "
        "`EXT_ICU4C_CXX_STANDARD' variable.")
    endif()
    message(FATAL_ERROR
      "An unsupported C++ compiler: "
      "CMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID}.")
  endif()

  if((NOT DEFINED EXT_ICU4C_WITH_ASAN)
      AND "${EXT_WITH_ASAN}")
    set(EXT_ICU4C_WITH_ASAN
      ON)
  endif()

  if("${EXT_ICU4C_WITH_ASAN}"
      OR "${EXT_ICU4C_WITH_PCSAN}"
      OR "${EXT_ICU4C_WITH_PSSAN}")
    list(APPEND
      EXT_ICU4C_CFLAGS
      -fsanitize=address)
    list(APPEND
      EXT_ICU4C_CXXFLAGS
      -fsanitize=address)
    list(APPEND
      EXT_ICU4C_LDFLAGS
      -fsanitize=address)
  endif()

  if((NOT DEFINED EXT_ICU4C_WITH_PCSAN)
      AND "${EXT_WITH_PCSAN}")
    set(EXT_ICU4C_WITH_PCSAN
      ON)
  endif()

  if("${EXT_ICU4C_WITH_PCSAN}")
    list(APPEND
      EXT_ICU4C_CFLAGS
      -fsanitize=pointer-compare)
    list(APPEND
      EXT_ICU4C_CXXFLAGS
      -fsanitize=pointer-compare)
    list(APPEND
      EXT_ICU4C_LDFLAGS
      -fsanitize=pointer-compare)
  endif()

  if((NOT DEFINED EXT_ICU4C_WITH_PSSAN)
      AND "${EXT_WITH_PSSAN}")
    set(EXT_ICU4C_WITH_PSSAN
      ON)
  endif()

  if("${EXT_ICU4C_WITH_PSSAN}")
    list(APPEND
      EXT_ICU4C_CFLAGS
      -fsanitize=pointer-subtract)
    list(APPEND
      EXT_ICU4C_CXXFLAGS
      -fsanitize=pointer-subtract)
    list(APPEND
      EXT_ICU4C_LDFLAGS
      -fsanitize=pointer-subtract)
  endif()

  if((NOT DEFINED EXT_ICU4C_WITH_TSAN)
      AND "${EXT_WITH_TSAN}")
    set(EXT_ICU4C_WITH_TSAN
      ON)
  endif()

  if("${EXT_ICU4C_WITH_TSAN}")
    list(APPEND
      EXT_ICU4C_CFLAGS
      -fsanitize=thread)
    list(APPEND
      EXT_ICU4C_CXXFLAGS
      -fsanitize=thread)
    list(APPEND
      EXT_ICU4C_LDFLAGS
      -fsanitize=thread)
  endif()

  if((NOT DEFINED EXT_ICU4C_WITH_LSAN)
      AND "${EXT_WITH_LSAN}")
    set(EXT_ICU4C_WITH_LSAN
      ON)
  endif()

  if("${EXT_ICU4C_WITH_LSAN}"
      AND (NOT "${EXT_ICU4C_WITH_ASAN}"))
    list(APPEND
      EXT_ICU4C_CFLAGS
      -fsanitize=leak)
    list(APPEND
      EXT_ICU4C_CXXFLAGS
      -fsanitize=leak)
    list(APPEND
      EXT_ICU4C_LDFLAGS
      -fsanitize=leak)
  endif()

  if((NOT DEFINED EXT_ICU4C_WITH_UBSAN)
      AND "${EXT_WITH_UBSAN}")
    set(EXT_ICU4C_WITH_UBSAN
      ON)
  endif()

  if("${EXT_ICU4C_WITH_UBSAN}")
    list(APPEND
      EXT_ICU4C_CFLAGS
      -fsanitize=undefined)
    list(APPEND
      EXT_ICU4C_CXXFLAGS
      -fsanitize=undefined)
    list(APPEND
      EXT_ICU4C_LDFLAGS
      -fsanitize=undefined)
  endif()

  if((NOT DEFINED EXT_ICU4C_WITH_LIBSTDCXX_DEBUG_MODE)
      AND "${EXT_WITH_LIBSTDCXX_DEBUG_MODE}")
    set(EXT_ICU4C_WITH_LIBSTDCXX_DEBUG_MODE
      ON)
  endif()

  if("${EXT_ICU4C_WITH_LIBSTDCXX_DEBUG_MODE}")
    list(APPEND
      EXT_ICU4C_CFLAGS
      -D_GLIBCXX_DEBUG
      -D_GLIBCXX_DEBUG_PEDANTIC)
    list(APPEND
      EXT_ICU4C_CXXFLAGS
      -D_GLIBCXX_DEBUG
      -D_GLIBCXX_DEBUG_PEDANTIC)
  endif()

  list(APPEND
    EXT_ICU4C_CONFIGURE_COMMAND
    "--cc=${CMAKE_C_COMPILER}")

  list(JOIN
    EXT_ICU4C_CFLAGS
    " "
    EXT_ICU4C_CFLAGS)
  list(APPEND
    EXT_ICU4C_CONFIGURE_COMMAND
    "--cflags=${EXT_ICU4C_CFLAGS}")

  list(APPEND
    EXT_ICU4C_CONFIGURE_COMMAND
    "--cxx=${CMAKE_CXX_COMPILER}")

  list(JOIN
    EXT_ICU4C_CXXFLAGS
    " "
    EXT_ICU4C_CXXFLAGS)
  list(APPEND
    EXT_ICU4C_CONFIGURE_COMMAND
    "--cxxflags=${EXT_ICU4C_CXXFLAGS}")

  list(JOIN
    EXT_ICU4C_LDFLAGS
    " "
    EXT_ICU4C_LDFLAGS)
  list(APPEND
    EXT_ICU4C_CONFIGURE_COMMAND
    "--ldflags=${EXT_ICU4C_LDFLAGS}")

  if(NOT DEFINED EXT_ICU4C_PREFIX)
    set(EXT_ICU4C_PREFIX
      "${CMAKE_CURRENT_BINARY_DIR}/icu4c-prefix"
      CACHE
      PATH
      "The prefix of the locations where ICU4C header and library files are installed.")
  endif()

  if(NOT IS_ABSOLUTE "${EXT_ICU4C_PREFIX}")
    message(FATAL_ERROR
      "A relative path `${EXT_ICU4C_PREFIX}' is specified for "
      "`EXT_ICU4C_PREFIX' variable, but is expected to be an absolute one.")
  endif()

  list(APPEND
    EXT_ICU4C_CONFIGURE_COMMAND
    --
    "--prefix=${EXT_ICU4C_PREFIX}")

  if("${EXT_ICU4C_USE_STATIC_LIBRARY}")
    list(APPEND
      EXT_ICU4C_CONFIGURE_COMMAND
      --disable-shared
      --enable-static)
  endif()

  set(EXT_ICU4C_BUILD_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/build"
    --build-dir=<BINARY_DIR>)

  set(EXT_ICU4C_INSTALL_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/install"
    --build-dir=<BINARY_DIR>)

  set(EXT_ICU4C_TEST_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/test"
    --build-dir=<BINARY_DIR>)

  ExternalProject_Add(ext_icu4c_build
    PREFIX icu4c-prefix
    DOWNLOAD_COMMAND ${EXT_ICU4C_DOWNLOAD_COMMAND}
    CONFIGURE_COMMAND ${EXT_ICU4C_CONFIGURE_COMMAND}
    BUILD_COMMAND ${EXT_ICU4C_BUILD_COMMAND}
    INSTALL_COMMAND ${EXT_ICU4C_INSTALL_COMMAND}
    TEST_COMMAND ${EXT_ICU4C_TEST_COMMAND}
    EXCLUDE_FROM_ALL ON)
else()
  add_custom_target(ext_icu4c_build)
endif()

add_library(ext_icu4c_library_search_path_option
  INTERFACE)
if(DEFINED EXT_ICU4C_PREFIX)
  if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    if((CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        OR (CMAKE_CXX_COMPILER_ID STREQUAL "GNU"))
      # Assume that Clang or GCC on Linux uses GNU linker or gold.
      target_link_libraries(ext_icu4c_library_search_path_option
        INTERFACE
        "-L${EXT_ICU4C_PREFIX}/lib")
    else()
      message(FATAL_ERROR
        "An unsupported C++ compiler: "
        "CMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID}.")
    endif()
  else()
    message(FATAL_ERROR
      "An unsupported system: CMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}.")
  endif()
endif()

foreach(deps "data" "uc;data" "i18n;uc;data" "io;i18n;uc;data"
    "tu;i18n;uc;data" "test;tu;i18n;uc;data")
  list(GET deps 0 lib)
  set(target_name
    "ext_icu4c_${lib}")

  add_library("${target_name}"
    INTERFACE)
  add_dependencies("${target_name}"
    ext_icu4c_build)
  if(DEFINED EXT_ICU4C_PREFIX)
    target_include_directories("${target_name}"
      INTERFACE
      "${EXT_ICU4C_PREFIX}/include")
  endif()
  target_link_libraries("${target_name}"
    INTERFACE
    ext_icu4c_library_search_path_option)

  if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    if((CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        OR (CMAKE_CXX_COMPILER_ID STREQUAL "GNU"))
      # Assume that Clang or GCC on Linux uses GNU linker or gold.
      if("${EXT_ICU4C_USE_STATIC_LIBRARY}")
        target_link_libraries("${target_name}"
          INTERFACE
          "-l:libicu4c${lib}.a")
      else()
        target_link_libraries("${target_name}"
          INTERFACE
          "-l:libicu4c${lib}.so")
      endif()
    else()
      message(FATAL_ERROR
        "An unsupported C++ compiler: "
        "CMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID}.")
    endif()
  else()
    message(FATAL_ERROR
      "An unsupported system: CMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}.")
  endif()

  list(LENGTH deps len)
  if(len GREATER_EQUAL 2)
    list(REMOVE_AT deps 0)
    foreach(dep IN LISTS deps)
      target_link_libraries("ext_icu4c_${lib}"
        INTERFACE
        "ext_icu4c_${dep}")
    endforeach()
  endif()
endforeach()
