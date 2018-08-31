#   This `CMakeLists.txt' file provides targets representing Boost header and
# library files. The Boost header and library files will be searched or
# installed, depending on variables set before this file is included.
#
#   If `EXT_BOOST_DOWNLOAD' is defined and evaluates to true, then a Boost
# source tree will be downloaded. The version to be downloaded is
#
#   - `EXT_BOOST_VERSION' if it is specified, otherwise
#   - the latest release version.
#
# The source tree will be located at
#
#   - `EXT_BOOST_SOURCE_DIR' if it is specified, otherwise
#   - `${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/src/boost'.
#
#   Next, if `EXT_BOOST_DOWLOAD' or `EXT_BOOST_SOURCE_DIR' is defined and
# evaluates to true, then Boost library files will be built. The following
# variables affect the configuration of the Boost library files to be built.
#
#   - EXT_BOOST_PREFIX: The prefix of the locations of header and library
#     files to install. The default value is
#     `${CMAKE_CURRENT_BINARY_DIR}/boost-prefix'.
#   - EXT_BOOST_LAYOUT: Build library files with `--layout' option. Allowed
#     values are `versioned', `tagged', and `system'. The default value is
#     `system'.
#   - EXT_BOOST_BUILD_ID: Build library files with `--buildid' option.
#   - EXT_BOOST_ENABLE_LIBRARIES: A list of Boost libraries to build. Allowed
#     values are
#       - atomic,
#       - chrono,
#       - container,
#       - context,
#       - contract,
#       - coroutine,
#       - date_time,
#       - exception,
#       - fiber,
#       - filesystem,
#       - graph,
#       - graph_parallel,
#       - iostreams,
#       - locale,
#       - log,
#       - math,
#       - mpi,
#       - program_options,
#       - python,
#       - random,
#       - regex,
#       - serialization,
#       - signals,
#       - stacktrace,
#       - system,
#       - test,
#       - thread,
#       - timer,
#       - type_erasure, and
#       - wave.
#   - EXT_BOOST_BUILD_TYPE: Specify the build type of library files. Allowed
#     values are `debug' and `release'. The default value is determined based
#     on the value of `CMAKE_BUILD_TYPE' variable.
#   - EXT_BOOST_CXX_STANDARD: Compile library files with the specified C++
#     language standard. Allowed values are `11', `14', and `17'. The default
#     value is determined based on the value of `CMAKE_CXX_STANDARD' variable.
#   - EXT_BOOST_USE_STATIC_LIBRARY: If this variable is defined and evaluates
#     to true, static library files will be built. Otherwise, shared ones will.
#   - EXT_BOOST_WITH_ASAN: If this variable is defined and evaluates to true,
#     library files will compile and link with AddressSanitizer
#     instrumentation.
#   - EXT_BOOST_WITH_PCSAN: If this variable is defined and evaluates to true,
#     library files will compile and link with instrumentation to check
#     pointer comparisons.
#   - EXT_BOOST_WITH_PSSAN: If this variable is defined and evaluates to true,
#     library files will compile and link with instrumentation to check
#     pointer subtractions.
#   - EXT_BOOST_WITH_TSAN: If this variable is defined and evaluates to true,
#     library files will compile and link with ThreadSanitizer
#     instrumentation.
#   - EXT_BOOST_WITH_LSAN: If this variable is defined and evaluates to true,
#     library files will compile and link with LeakSanitizer instrumentation.
#   - EXT_BOOST_WITH_UBSAN: If this variable is defined and evaluates to true,
#     library files will compile and link with UndefinedBehaviorSanitizer
#     instrumentation.
#   - EXT_BOOST_WITH_LIBSTDCXX_DEBUG_MODE: If this variable is defined and
#     evaluates to true, library files will compile with libstdc++ debug mode.
#
#   Finally, the following targets represent the Boost header and library
# files:
#
#   - ext_boost_headers
#   - ext_boost_atomic
#   - ext_boost_container
#   - ext_boost_context
#   - ext_boost_date_time
#   - ext_boost_exception
#   - ext_boost_iostreams
#   - ext_boost_math_c99f
#   - ext_boost_math_c99l
#   - ext_boost_math_c99
#   - ext_boost_program_options
#   - ext_boost_regex
#   - ext_boost_graph
#   - ext_boost_serialization
#   - ext_boost_signals
#   - ext_boost_system
#   - ext_boost_chrono
#   - ext_boost_contract
#   - ext_boost_filesystem
#   - ext_boost_fiber
#   - ext_boost_random
#   - ext_boost_test_exec_monitor
#   - ext_boost_thread
#   - ext_boost_coroutine
#   - ext_boost_locale
#   - ext_boost_log
#   - ext_boost_log_setup
#   - ext_boost_timer
#   - ext_boost_prg_exec_monitor
#   - ext_boost_type_erasure
#   - ext_boost_unit_test_framework
#   - ext_boost_wave
#   - ext_boost_wserialization
#
# If `EXT_BOOST_DOWLOAD' or `EXT_BOOST_SOURCE_DIR' is defined and evaluates to
# true, then the above targets are IMPORTED targets, which will have been
# built as described above. Otherwise, the above targets are interface
# libraries that just represent compiler and linker flags necessary to include
# header files and link library files. If the installed header or library
# files are not in a standard location, you can explicitly specify
# `EXT_BOOST_PREFIX' to tell the compiler and linker the directories where
# they are installed.

cmake_minimum_required(VERSION 3.12.0)
include(ExternalProject)

if((DEFINED EXT_BOOST_SOURCE_DIR)
    AND (NOT IS_ABSOLUTE "${EXT_BOOST_SOURCE_DIR}"))
  message(FATAL_ERROR
    "A relative path `${EXT_BOOST_SOURCE_DIR}' is specified for "
    "`EXT_BOOST_SOURCE_DIR' variable, but is expected to be an absolute one.")
endif()

if("${EXT_BOOST_DOWNLOAD}")
  set(EXT_BOOST_DOWNLOAD_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/download")
  if(DEFINED EXT_BOOST_VERSION)
    list(APPEND
      EXT_BOOST_DOWNLOAD_COMMAND
      "--version=${EXT_BOOST_VERSION}")
  endif()
  if(NOT DEFINED EXT_BOOST_SOURCE_DIR)
    set(EXT_BOOST_SOURCE_DIR
      "${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/src/boost")
  endif()
  list(APPEND
    EXT_BOOST_DOWNLOAD_COMMAND
    "--source-dir=${EXT_BOOST_SOURCE_DIR}"
    --clobber-source-dir)
else()
  set(EXT_BOOST_DOWNLOAD_COMMAND
    true)
endif()

if(DEFINED EXT_BOOST_SOURCE_DIR)
  if(NOT DEFINED EXT_BOOST_PREFIX)
    set(EXT_BOOST_PREFIX
      "${CMAKE_CURRENT_BINARY_DIR}/boost-prefix"
      CACHE
      PATH
      "The prefix of the locations where Boost header and library files are installed.")
  endif()

  if(NOT IS_ABSOLUTE "${EXT_BOOST_PREFIX}")
    message(FATAL_ERROR
      "A relative path `${EXT_BOOST_PREFIX}' is specified for "
      "`EXT_BOOST_PREFIX' variable, but is expected to be an absolute one.")
  endif()

  set(EXT_BOOST_BUILD_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/build"
    "--source-dir=${EXT_BOOST_SOURCE_DIR}"
    "--prefix=${EXT_BOOST_PREFIX}"
    --
    "--build-type=minimal"
    "--build-dir=<BINARY_DIR>")

  if(NOT DEFINED EXT_BOOST_LAYOUT)
    # Do nothing.
  elseif("${EXT_BOOST_LAYOUT}" STREQUAL "versioned")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      --layout=versioned)
  elseif("${EXT_BOOST_LAYOUT}" STREQUAL "tagged")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      --layout=tagged)
  elseif("${EXT_BOOST_LAYOUT}" STREQUAL "system")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      --layout=system)
  else()
    message(FATAL_ERROR
      "An invalid value: EXT_BOOST_LAYOUT=${EXT_BOOST_LAYOUT}.")
  endif()

  if(DEFINED EXT_BOOST_BUILD_ID)
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "--buildid=${EXT_BOOST_BUILD_ID}")
  endif()

  foreach(library IN LISTS EXT_BOOST_ENABLE_LIBRARIES)
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "--with-${library}")
    if(((library STREQUAL "regex")
          OR (library STREQUAL "locale"))
        AND (NOT "${EXT_BOOST_WITHOUT_ICU4C}"))
      set(EXT_BOOST_DEPENDS_ON_ICU4C ON)
    endif()
    if((library STREQUAL "stacktrace")
        AND (NOT "${EXT_BOOST_WITHOUT_LIBBACKTRACE}"))
      set(EXT_BOOST_DEPENDS_ON_LIBBACKTRACE ON)
    endif()
  endforeach()

  if("${EXT_BOOST_DEPENDS_ON_ICU4C}")
    if(DEFINED EXT_ICU4C_PREFIX)
      list(APPEND
        EXT_BOOST_BUILD_COMMAND
        "-sICU_PATH=${EXT_ICU4C_PREFIX}")
    endif()
    add_library(ext_boost_icu4c_build
      ALIAS
      ext_icu4c_build)
  else()
    add_custom_target(ext_boost_icu4c_build)
  endif()

  if("${EXT_BOOST_DEPENDS_ON_LIBBACKTRACE}")
    if(DEFINED EXT_LIBBACKTRACE_PREFIX)
      list(APPEND
        EXT_BOOST_BUILD_COMMAND
        "-sLIBBACKTRACE_PATH=${EXT_LIBBACKTRACE_PREFIX}")
    endif()
    add_custom_target(ext_boost_libbacktrace_build)
    add_dependencies(ext_boost_libbacktrace_build
      ext_libbacktrace_build)
  else()
    add_custom_target(ext_boost_libbacktrace_build)
  endif()

  if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      toolset=clang)
  elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      toolset=gcc)
  else()
    message(FATAL_ERROR
      "An unsupported C++ compiler: "
      "CMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID}")
  endif()

  if(NOT DEFINED EXT_BOOST_CXX_STANDARD)
    if(NOT DEFINED EXT_CXX_STANDARD)
      # Do nothing.
    elseif(EXT_CXX_STANDARD STREQUAL "11")
      set(EXT_BOOST_CXX_STANDARD
        11)
    elseif(EXT_CXX_STANDARD STREQUAL "14")
      set(EXT_BOOST_CXX_STANDARD
        14)
    elseif(EXT_CXX_STANDARD STREQUAL "17")
      set(EXT_BOOST_CXX_STANDARD
        17)
    else()
      message(FATAL_ERROR
        "An invalid value: EXT_CXX_STANDARD=${EXT_CXX_STANDARD}.")
    endif()
  endif()

  if((CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
      OR (CMAKE_CXX_COMPILER_ID STREQUAL "GNU"))
    if(NOT DEFINED EXT_BOOST_CXX_STANDARD)
      # Do nothing.
    elseif(EXT_BOOST_CXX_STANDARD STREQUAL "11")
      list(APPEND
        EXT_BOOST_BUILD_COMMAND
        cxxflags=-std=c++11)
    elseif(EXT_BOOST_CXX_STANDARD STREQUAL "14")
      list(APPEND
        EXT_BOOST_BUILD_COMMAND
        cxxflags=-std=c++14)
    elseif(EXT_BOOST_CXX_STANDARD STREQUAL "17")
      list(APPEND
        EXT_BOOST_BUILD_COMMAND
        cxxflags=-std=c++17)
    else()
      message(FATAL_ERROR
        "An invalid value: EXT_BOOST_CXX_STANDARD=${EXT_BOOST_CXX_STANDARD}.")
    endif()
  else()
    message(FATAL_ERROR
      "An unsupported C++ compiler: "
      "CMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID}.")
  endif()

  if(NOT DEFINED EXT_BOOST_BUILD_TYPE)
    set(EXT_BOOST_BUILD_TYPE
      "${EXT_BUILD_TYPE}")
  endif()

  if(EXT_BOOST_BUILD_TYPE STREQUAL "debug")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      variant=debug)
  elseif(EXT_BOOST_BUILD_TYPE STREQUAL "release")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      variant=release)
  else()
    message(FATAL_ERROR
      "An invalid value: EXT_BOOST_BUILD_TYPE=${EXT_BOOST_BUILD_TYPE}.")
  endif()

  if("${EXT_BOOST_USE_STATIC_LIBRARY}")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      link=static)
  else()
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      link=shared)
  endif()

  list(APPEND
    EXT_BOOST_BUILD_COMMAND
    threading=multi)

  if((NOT DEFINED EXT_BOOST_WITH_ASAN)
      AND "${EXT_WITH_ASAN}")
    set(EXT_BOOST_WITH_ASAN
      ON)
  endif()

  if("${EXT_BOOST_WITH_ASAN}"
      OR "${EXT_BOOST_WITH_PCSAN}"
      OR "${EXT_BOOST_WITH_PSSAN}")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      cflags=-fsanitize=address
      cxxflags=-fsanitize=address
      linkflags=-fsanitize=address)
  endif()

  if((NOT DEFINED EXT_BOOST_WITH_PCSAN)
      AND "${EXT_WITH_PCSAN}")
    set(EXT_BOOST_WITH_PCSAN
      ON)
  endif()

  if("${EXT_BOOST_WITH_PCSAN}")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      cflags=-fsanitize=pointer-compare
      cxxflags=-fsanitize=pointer-compare
      linkflags=-fsanitize=pointer-compare)
  endif()

  if((NOT DEFINED EXT_BOOST_WITH_PSSAN)
      AND "${EXT_WITH_PSSAN}")
    set(EXT_BOOST_WITH_PSSAN
      ON)
  endif()

  if("${EXT_BOOST_WITH_PSSAN}")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      cflags=-fsanitize=pointer-subtract
      cxxflags=-fsanitize=pointer-subtract
      linkflags=-fsanitize=pointer-subtract)
  endif()

  if((NOT DEFINED EXT_BOOST_WITH_TSAN)
      AND "${EXT_WITH_TSAN}")
    set(EXT_BOOST_WITH_TSAN
      ON)
  endif()

  if("${EXT_BOOST_WITH_TSAN}")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      cflags=-fsanitize=thread
      cxxflags=-fsanitize=thread
      linkflags=-fsanitize=thread)
  endif()

  if((NOT DEFINED EXT_BOOST_WITH_LSAN)
      AND "${EXT_WITH_LSAN}")
    set(EXT_BOOST_WITH_LSAN
      ON)
  endif()

  if("${EXT_BOOST_WITH_LSAN}")
    if(NOT "${EXT_BOOST_WITH_ASAN}")
      list(APPEND
        EXT_BOOST_BUILD_COMMAND
        cflags=-fsanitize=leak
        cxxflags=-fsanitize=leak
        linkflags=-fsanitize=leak)
    endif()
  endif()

  if((NOT DEFINED EXT_BOOST_WITH_UBSAN)
      AND "${EXT_WITH_UBSAN}")
    set(EXT_BOOST_WITH_UBSAN
      ON)
  endif()

  if("${EXT_BOOST_WITH_UBSAN}")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      cflags=-fsanitize=undefined
      cxxflags=-fsanitize=undefined
      linkflags=-fsanitize=undefined)
  endif()

  if((NOT DEFINED EXT_BOOST_WITH_LIBSTDCXX_DEBUG_MODE)
      AND "${EXT_WITH_LIBSTDCXX_DEBUG_MODE}")
    set(EXT_BOOST_WITH_LIBSTDCXX_DEBUG_MODE
      ON)
  endif()

  if("${EXT_BOOST_WITH_LIBSTDCXX_DEBUG_MODE}")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      define=_GLIBCXX_DEBUG
      define=_GLIBCXX_DEBUG_PEDANTIC)
  endif()

  ExternalProject_Add(ext_boost_build
    PREFIX "${CMAKE_CURRENT_BINARY_DIR}/boost-prefix"
    DOWNLOAD_COMMAND ${EXT_BOOST_DOWNLOAD_COMMAND}
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ${EXT_BOOST_BUILD_COMMAND}
    INSTALL_COMMAND ""
    TEST_COMMAND ""
    DEPENDS ext_boost_libbacktrace_build ext_boost_icu4c_build
    EXCLUDE_FROM_ALL ON)
else()
  add_custom_target(ext_boost_build)
endif()

add_library(ext_boost_headers
  INTERFACE)
add_dependencies(ext_boost_headers
  ext_boost_build)
if(DEFINED EXT_BOOST_PREFIX)
  target_include_directories(ext_boost_headers
    INTERFACE
    "${EXT_BOOST_PREFIX}/include")
endif()

add_library(ext_boost_library_search_path_option
  INTERFACE)
if(DEFINED EXT_BOOST_PREFIX)
  if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    if((CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        OR (CMAKE_CXX_COMPILER_ID STREQUAL "GNU"))
      # Assume that Clang or GCC on Linux uses GNU linker or gold.
      target_link_libraries(ext_boost_library_search_path_option
        INTERFACE
        "-L${EXT_BOOST_PREFIX}/lib")
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

set(EXT_BOOST_LIBRARY_SUFFIX
  "")
if("${EXT_BOOST_LAYOUT}" STREQUAL "versioned")
  # TODO: Compiler version tag, e.g., `-gcc82'.
endif()
if(("${EXT_BOOST_LAYOUT}" STREQUAL "tagged")
    OR ("${EXT_BOOST_LAYOUT}" STREQUAL "versioned"))
  set(EXT_BOOST_LIBRARY_SUFFIX
    "${EXT_BOOST_LIBRARY_SUFFIX}-mt")
  # TODO: `s'
  if("${EXT_BOOST_BUILD_TYPE}" STREQUAL "debug")
    set(EXT_BOOST_LIBRARY_SUFFIX
      "${EXT_BOOST_LIBRARY_SUFFIX}-g")
  endif()
endif()
if("${EXT_BOOST_LAYOUT}" STREQUAL "versioned")
  # TODO: Architecture and address model tag, e.g., `-x64'.
  # TODO: Boost version tag, e.g., `-1_68'.
endif()
if(DEFINED EXT_BOOST_BUILD_ID)
  set(EXT_BOOST_LIBRARY_SUFFIX
    "${EXT_BOOST_LIBRARY_SUFFIX}-${EXT_BOOST_BUILD_ID}")
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  if("${EXT_BOOST_USE_STATIC_LIBRARY}")
    set(EXT_BOOST_LIBRARY_SUFFIX
      "${EXT_BOOST_LIBRARY_SUFFIX}.a")
  else()
    set(EXT_BOOST_LIBRARY_SUFFIX
      "${EXT_BOOST_LIBRARY_SUFFIX}.so")
  endif()
else()
  message(FATAL_ERROR
    "An unsupported system: CMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}.")
endif()

foreach(libs "atomic;ATOMIC" "container;CONTAINER" "context;CONTEXT"
    "date_time;DATE_TIME" "exception" "iostreams;IOSTREAMS" "math_c99f"
    "math_c99l" "math_c99" "program_options;PROGRAM_OPTIONS" "regex;REGEX"
    "graph;GRAPH;regex" "serialization;SERIALIZATION" "signals;SIGNALS"
    "stacktrace_addr2line;STACKTRACE" "stacktrace_backtrace;STACKTRACE"
    "stacktrace_basic;STACKTRACE" "stacktrace_noop;STACKTRACE" "system;SYSTEM"
    "chrono;CHRONO;system" "contract;CONTRACT;system"
    "filesystem;FILESYSTEM;system" "fiber;FIBERS;context;filesystem;system"
    "random;RANDOM;system" "test_exec_monitor;TEST" "thread;THREAD;system"
    "coroutine;COROUTINES;context;chrono;thread;system"
    "locale;LOCALE;chrono;thread;system"
    "log;LOG;atomic;date_time;regex;chrono;filesystem;thread;system"
    "log_setup;LOG;log;atomic;date_time;regex;chrono;filesystem;thread;system"
    "timer;TIMER;chrono;system" "prg_exec_monitor;TEST;timer;system"
    "type_erasure;TYPE_ERASURE;chrono;thread;system"
    "unit_test_framework;TEST;timer;system"
    "wave;WAVE;date_time;chrono;filesystem;thread;system"
    "wserialization;SERIALIZATION;serialization")
  list(GET libs 0 lib)
  add_library("ext_boost_${lib}"
    INTERFACE)

  list(LENGTH libs len)
  if(len GREATER_EQUAL 2)
    list(GET libs 1 LIB)
    if(NOT "${EXT_BOOST_USE_STATIC_LIBRARY}")
      target_compile_definitions("ext_boost_${lib}"
        INTERFACE
        "BOOST_${LIB}_DYN_LINK")
    endif()
  endif()

  target_link_libraries("ext_boost_${lib}"
    INTERFACE
    ext_boost_headers)

  target_link_libraries("ext_boost_${lib}"
    INTERFACE
    ext_boost_library_search_path_option
    "-l:libboost_${lib}${EXT_BOOST_LIBRARY_SUFFIX}")

  if(len GREATER_EQUAL 3)
    list(REMOVE_AT libs 0 1)
    foreach(dep IN LISTS libs)
      target_link_libraries("ext_boost_${lib}"
        INTERFACE
        "ext_boost_${dep}")
    endforeach()
  endif()
endforeach()

if(NOT "${EXT_BOOST_WITHOUT_ICU4C}")
  target_link_libraries(ext_boost_regex
    INTERFACE
    ext_icu4c_i18n)
  target_link_libraries(ext_boost_locale
    INTERFACE
    ext_icu4c_i18n)
endif()

target_compile_definitions(ext_boost_stacktrace_addr2line
  INTERFACE
  BOOST_STACKTRACE_USE_ADDR2LINE)
target_link_libraries(ext_boost_stacktrace_addr2line
  INTERFACE
  -ldl
  -rdynamic)

target_compile_definitions(ext_boost_stacktrace_backtrace
  INTERFACE
  BOOST_STACKTRACE_USE_BACKTRACE)
target_link_libraries(ext_boost_stacktrace_backtrace
  INTERFACE
  ext_libbacktrace
  -ldl
  -rdynamic)

target_link_libraries(ext_boost_stacktrace_basic
  INTERFACE
  -ldl
  -rdynamic)

target_compile_definitions(ext_boost_stacktrace_noop
  INTERFACE
  BOOST_STACKTRACE_USE_NOOP)
