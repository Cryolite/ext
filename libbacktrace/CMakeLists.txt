cmake_minimum_required(VERSION 3.12.0)
include(ExternalProject)

if((DEFINED EXT_LIBBACKTRACE_SOURCE_DIR)
    AND (NOT IS_ABSOLUTE "${EXT_LIBBACKTRACE_SOURCE_DIR}"))
  message(FATAL_ERROR
    "A relative path `${EXT_LIBBACKTRACE_SOURCE_DIR}' is specified for "
    "`EXT_LIBBACKTRACE_SOURCE_DIR' variable, but is expected to be an "
    "absolute one.")
endif()

if("${EXT_LIBBACKTRACE_DOWNLOAD}")
  set(EXT_LIBBACKTRACE_DOWNLOAD_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/download")
  if(NOT DEFINED EXT_LIBBACKTRACE_SOURCE_DIR)
    set(EXT_LIBBACKTRACE_SOURCE_DIR
      "${CMAKE_CURRENT_BINARY_DIR}/libbacktrace-prefix/src/libbacktrace")
  endif()
  list(APPEND
    EXT_LIBBACKTRACE_DOWNLOAD_COMMAND
    "--source-dir=${EXT_LIBBACKTRACE_SOURCE_DIR}"
    --clobber-source-dir)
else()
  set(EXT_LIBBACKTRACE_DOWNLOAD_COMMAND
    true)
endif()

if(DEFINED EXT_LIBBACKTRACE_SOURCE_DIR)
  if(NOT DEFINED EXT_LIBBACKTRACE_PREFIX)
    set(EXT_LIBBACKTRACE_PREFIX
      "${CMAKE_CURRENT_BINARY_DIR}/libbacktrace-prefix"
      CACHE
      PATH
      "The prefix of the locations where libbacktrace header and library files are installed.")
  endif()

  if(NOT IS_ABSOLUTE "${EXT_LIBBACKTRACE_PREFIX}")
    message(FATAL_ERROR
      "A relative path `${EXT_LIBBACKTRACE_PREFIX}' is specified for "
      "`EXT_LIBBACKTRACE_PREFIX' variable, but is expected to be an absolute "
      "one.")
  endif()

  set(EXT_LIBBACKTRACE_CONFIGURE_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/configure"
    "--source-dir=${EXT_LIBBACKTRACE_SOURCE_DIR}"
    --build-dir=<BINARY_DIR>
    --clobber-build-dir
    --
    "--prefix=${EXT_LIBBACKTRACE_PREFIX}")

  if("${EXT_LIBBACKTRACE_USE_STATIC_LIBRARY}")
    list(APPEND
      EXT_LIBBACKTRACE_CONFIGURE_COMMAND
      --disable-shared)
  else()
    list(APPEND
      EXT_LIBBACKTRACE_CONFIGURE_COMMAND
      --disable-static)
  endif()

  list(APPEND
    EXT_LIBBACKTRACE_CONFIGURE_COMMAND
    "CC=${CMAKE_C_COMPILER}")

  set(EXT_LIBBACKTRACE_BUILD_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/build"
    --build-dir=<BINARY_DIR>)

  set(EXT_LIBBACKTRACE_INSTALL_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/install"
    --build-dir=<BINARY_DIR>)

  set(EXT_LIBBACKTRACE_TEST_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/test"
    --build-dir=<BINARY_DIR>)

  ExternalProject_Add(ext_libbacktrace_build
    PREFIX libbacktrace-prefix
    DOWNLOAD_COMMAND ${EXT_LIBBACKTRACE_DOWNLOAD_COMMAND}
    CONFIGURE_COMMAND ${EXT_LIBBACKTRACE_CONFIGURE_COMMAND}
    BUILD_COMMAND ${EXT_LIBBACKTRACE_BUILD_COMMAND}
    INSTALL_COMMAND ${EXT_LIBBACKTRACE_INSTALL_COMMAND}
    TEST_COMMAND ${EXT_LIBBACKTRACE_TEST_COMMAND}
    EXCLUDE_FROM_ALL ON)
else()
  add_custom_target(ext_libbacktrace_build)
endif()

add_library(ext_libbacktrace
  INTERFACE)
add_dependencies(ext_libbacktrace
  ext_libbacktrace_build)
if(DEFINED EXT_LIBBACKTRACE_PREFIX)
  target_include_directories(ext_libbacktrace
    INTERFACE
    "${EXT_LIBBACKTRACE_PREFIX}/include")
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  if((CMAKE_C_COMPILER_ID STREQUAL "Clang")
      OR (CMAKE_C_COMPILER_ID STREQUAL "GNU"))
    # Assume that Clang or GCC on Linux uses GNU linker or gold.
    if(DEFINED EXT_LIBBACKTRACE_PREFIX)
      target_link_libraries(ext_libbacktrace
        INTERFACE
        "-L${EXT_LIBBACKTRACE_PREFIX}/lib")
    endif()
    if("${EXT_LIBBACKTRACE_USE_STATIC_LIBRARY}")
      target_link_libraries(ext_libbacktrace
        INTERFACE
        -l:libbacktrace.a)
    else()
      target_link_libraries(ext_libbacktrace
        INTERFACE
        -l:libbacktrace.so)
    endif()
  else()
    message(FATAL_ERROR
      "An unsupported C compiler: "
      "CMAKE_C_COMPILER_ID=${CMAKE_C_COMPILER_ID}.")
  endif()
else()
  message(FATAL_ERROR
    "An unsupported system: CMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}.")
endif()
