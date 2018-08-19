#   This `CMakeLists.txt' file provides targets representing Boost header and
# library files. The Boost header and library files will be searched or
# installed, depending on variables set before this file is included.
#
#   If `EXT_BOOST_DOWNLOAD' is set and evaluates to true, then a Boost source
# tree will be downloaded. The version to be downloaded is
#
#   - `EXT_BOOST_VERSION' if it is specified, otherwise
#   - the latest release version.
#
# The source tree will be located at
#
#   - `EXT_BOOST_SOURCE_DIR' if it is specified, otherwise
#   - `${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/src/boost'.
#
#   Next, if `EXT_BOOST_DOWLOAD' or `EXT_BOOST_SOURCE_DIR' is set and
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
#   - EXT_BOOST_ENABLE_LIBRARIES: A list of Boost libraries to build.
#   - EXT_BOOST_CXX_STANDARD: Compile library files with the specified C++
#     language standard. Allowed values are `11', `14', and `17'. The default
#     value is determined based on the value of `CMAKE_CXX_STANDARD' variable.
#   - EXT_BOOST_BUILD_TYPE: Specify the build type of library files. Allowed
#     values are `debug' and `release'. The default value is determined based
#     on the value of `CMAKE_BUILD_TYPE' variable.
#   - EXT_BOOST_SHARED_LIBS: If this variable is set and evaluates to true,
#     shared library files will be built. Otherwise, static library files
#     will. The default value is determined based on the value of
#     `BUILD_SHARED_LIBS' variable.
#   - EXT_BOOST_WITH_ASAN: If this variable is set and evaluates to true,
#     library files will compile and link with AddressSanitizer
#     instrumentation.
#   - EXT_BOOST_WITH_UBSAN: If this variable is set and evaluates to true,
#     library files will compile and link with UndefinedSanitizer
#     instrumentation.
#   - EXT_BOOST_WITH_TSAN: If this variable is set and evaluates to true,
#     library files will compile and link with ThreadSanitizer
#     instrumentation.
#   - EXT_BOOST_WITH_LIBSTDCXX_DEBUG_MODE: If this variable is set and
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
# If `EXT_BOOST_DOWLOAD' or `EXT_BOOST_SOURCE_DIR' is set and evaluates to
# true, then the above targets are IMPORTED targets, which will have been
# built as described above. Otherwise, the above targets are interface
# libraries that just represent compiler and linker flags necessary to include
# header files and link library files. If the installed header or library
# files are not in a standard location, you can explicitly specify
# `EXT_BOOST_PREFIX' to tell the compiler and linker the directories where
# they are installed.

include(ExternalProject)

if(EXT_BOOST_DOWNLOAD)
  set(EXT_BOOST_DOWNLOAD_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/download")
  if(EXT_BOOST_VERSION)
    list(APPEND
      EXT_BOOST_DOWNLOAD_COMMAND
      "--version=${EXT_BOOST_VERSION}")
  endif()
  if(NOT EXT_BOOST_SOURCE_DIR)
    set(EXT_BOOST_SOURCE_DIR
      "${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/src/boost"
      CACHE
      PATH
      "Boost source directory.")
  endif()
  list(APPEND
    EXT_BOOST_DOWNLOAD_COMMAND
    "--source-dir=${EXT_BOOST_SOURCE_DIR}"
    "--clobber-source-dir")
else()
  set(EXT_BOOST_DOWNLOAD_COMMAND
    "true")
endif()

if(EXT_BOOST_SOURCE_DIR)
  set(EXT_BOOST_BUILD_COMMAND
    "${CMAKE_CURRENT_SOURCE_DIR}/build"
    "--source-dir=${EXT_BOOST_SOURCE_DIR}")
  if(NOT EXT_BOOST_PREFIX)
    set(EXT_BOOST_PREFIX
      "${CMAKE_CURRENT_BINARY_DIR}/boost-prefix"
      CACHE
      PATH
      "Boost install location.")
  endif()
  list(APPEND
    EXT_BOOST_BUILD_COMMAND
    "--prefix=${EXT_BOOST_PREFIX}"
    "--"
    "--build-type=minimal"
    "--build-dir=<BINARY_DIR>")
  if(NOT EXT_BOOST_LAYOUT)
    set(EXT_BOOST_LAYOUT
      "--layout=system")
  else()
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "--layout=${EXT_BOOST_LAYOUT}")
  endif()
  if(EXT_BOOST_BUILD_ID)
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "--buildid=${EXT_BOOST_BUILD_ID}")
  endif()
  foreach(library IN LISTS EXT_BOOST_ENABLE_LIBRARIES)
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "--with-${library}")
  endforeach()
  if(EXT_ICU4C_PREFIX)
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "-sICU_PATH=${EXT_ICU4C_PREFIX}")
  endif()
  if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "toolset=clang")
  elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "toolset=gcc")
  else()
    message(FATAL_ERRRO
      "An unsupported compiler: CMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID}")
  endif()
  if(NOT EXT_BOOST_CXX_STANDARD)
    if(CMAKE_CXX_STANDARD STREQUAL "11")
      set(EXT_BOOST_CXX_STANDARD
        "11"
        CACHE
        STRING
        "The C++ language standard.")
    elseif(CMAKE_CXX_STANDARD STREQUAL "14")
      set(EXT_BOOST_CXX_STANDARD
        "14"
        CACHE
        STRING
        "The C++ language standard.")
    elseif(CMAKE_CXX_STANDARD STREQUAL "17")
      set(EXT_BOOST_CXX_STANDARD
        "17"
        CACHE
        STRING
        "The C++ language standard.")
    else()
      message(FATAL_ERROR
        "An unsupported c++ language standard: CMAKE_CXX_STANDARD=${CMAKE_CXX_STANDARD}")
    endif()
  endif()
  if(EXT_BOOST_CXX_STANDARD STREQUAL "11")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "cxxflags=-std=c++11")
  elseif(EXT_BOOST_CXX_STANDARD STREQUAL "14")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "cxxflags=-std=c++14")
  elseif(EXT_BOOST_CXX_STANDARD STREQUAL "17")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "cxxflags=-std=c++17")
  else()
    message(FATAL_ERROR
      "An invalid value `${EXT_BOOST_CXX_STANDARD}' for `EXT_BOOST_CXX_STANDARD' variable.")
  endif()
  if(NOT EXT_BOOST_BUILD_TYPE)
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
      set(EXT_BOOST_BUILD_TYPE
        "debug"
        CACHE
        STRING
        "The build type of Boost library files (allowed values are `debug' and `release').")
    elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
      set(EXT_BOOST_BUILD_TYPE
        "release"
        CACHE
        STRING
        "The build type of Boost library files (allowed values are `debug' and `release').")
    else()
      message(FATAL_ERROR
        "An unsupported build type: CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}")
    endif()
  endif()
  if(EXT_BOOST_BUILD_TYPE STREQUAL "debug")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "variant=debug")
  elseif(EXT_BOOST_BUILD_TYPE STREQUAL "release")
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "variant=release")
  else()
    message(FATAL_ERROR
      "An invalid value `${EXT_BOOST_BUILD_TYPE}' for `EXT_BOOST_BUILD_TYPE' variable.")
  endif()
  if(NOT EXT_BOOST_SHARED_LIBS AND BUILD_SHARED_LIBS)
    set(EXT_BOOST_SHARED_LIBS
      "ON")
  endif()
  if(EXT_BOOST_SHARED_LIBS)
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "link=shared")
  else()
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "link=static")
  endif()
  list(APPEND
    EXT_BOOST_BUILD_COMMAND
    "threading=multi")
  if(EXT_BOOST_WITH_ASAN)
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "cflags=-fsanitize=address"
      "cxxflags=-fsanitize=address"
      "linkflags=-fsanitize=address")
  endif()
  if(EXT_BOOST_WITH_UBSAN)
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "cflags=-fsanitize=undefined"
      "cxxflags=-fsanitize=undefined"
      "linkflags=-fsanitize=undefined")
  endif()
  if(EXT_BOOST_WITH_TSAN)
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "cflags=-fsanitize=thread"
      "cxxflags=-fsanitize=thread"
      "linkflags=-fsanitize=thread")
  endif()
  if(EXT_BOOST_WITH_LIBSTDCXX_DEBUG_MODE)
    list(APPEND
      EXT_BOOST_BUILD_COMMAND
      "define=_GLIBCXX_DEBUG"
      "define=_GLIBCXX_DEBUG_PEDANTIC")
  endif()
  if(EXT_ICU4C_SOURCE_DIR)
    ExternalProject_Add(ext_boost
      PREFIX "${CMAKE_CURRENT_BINARY_DIR}/boost-prefix"
      DOWNLOAD_COMMAND ${EXT_BOOST_DOWNLOAD_COMMAND}
      CONFIGURE_COMMAND ""
      BUILD_COMMAND ${EXT_BOOST_BUILD_COMMAND}
      INSTALL_COMMAND ""
      TEST_COMMAND ""
      DEPENDS ext_icu4c)
  else()
    ExternalProject_Add(ext_boost
      PREFIX "${CMAKE_CURRENT_BINARY_DIR}/boost-prefix"
      DOWNLOAD_COMMAND ${EXT_BOOST_DOWNLOAD_COMMAND}
      CONFIGURE_COMMAND ""
      BUILD_COMMAND ${EXT_BOOST_BUILD_COMMAND}
      INSTALL_COMMAND ""
      TEST_COMMAND "")
  endif()

  add_library(ext_boost_headers
    INTERFACE)
  target_include_directories(ext_boost_headers
    INTERFACE
    "${EXT_BOOST_PREFIX}/include")
  add_dependencies(ext_boost_headers
    ext_boost)

  if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    set(EXT_BOOST_LIBRARY_SUFFIX "")
    if(EXT_BOOST_LAYOUT STREQUAL "versioned")
      # TODO: Compiler version tag, e.g., `-gcc82'.
    endif()
    if(EXT_BOOST_LAYOUT STREQUAL "tagged"
        OR EXT_BOOST_LAYOUT STREQUAL "versioned")
      set(EXT_BOOST_LIBRARY_SUFFIX
        "${EXT_BOOST_LIBRARY_SUFFIX}-mt")
      # TODO: `s'
      if(EXT_BOOST_LIBRARY_SUFFIX STREQUAL "debug")
        set(EXT_BOOST_LIBRARY_SUFFIX
          "${EXT_BOOST_LIBRARY_SUFFIX}-g")
      endif()
    endif()
    if(EXT_BOOST_LAYOUT STREQUAL "versioned")
      # TODO: Architecture and address model tag, e.g., `-x64'.
      # TODO: Boost version tag, e.g., `-1_68'.
    endif()
    if(EXT_BOOST_BUILD_ID)
      set(EXT_BOOST_LIBRARY_SUFFIX
        "${EXT_BOOST_LIBRARY_SUFFIX}-${EXT_BOOST_BUILD_ID}")
    endif()
    if(EXT_BOOST_SHARED_LIBS)
      set(EXT_BOOST_LIBRARY_TYPE
        "SHARED")
      set(EXT_BOOST_LIBRARY_SUFFIX
        "${EXT_BOOST_LIBRARY_SUFFIX}.so")
    else()
      set(EXT_BOOST_LIBRARY_TYPE
        "STATIC")
      set(EXT_BOOST_LIBRARY_SUFFIX
        "${EXT_BOOST_LIBRARY_SUFFIX}.a")
    endif()

    foreach(spec "atomic;ATOMIC" "container;CONTAINER" "context;CONTEXT"
        "date_time;DATE_TIME" "iostreams;IOSTREAMS" "math_c99f" "math_c99l"
        "math_c99" "program_options;PROGRAM_OPTIONS" "regex;REGEX"
        "graph;GRAPH;regex" "serialization;SERIALIZATION" "signals;SIGNALS"
        "system;SYSTEM" "chrono;CHRONO;system" "contract;CONTRACT;system"
        "filesystem;FILESYSTEM;system"
        "fiber;FIBERS;context;system;filesystem" "random;RANDOM;system"
        "text_exec_monitor;TEST" "thread;THREAD;system"
        "coroutine;COROUTINES;context;system;chrono;thread"
        "locale;LOCALE;system;chrono;thread"
        "log;LOG;atomic;date_time;regex;system;chrono;filesystem;thread"
        "log_setup;LOG;atomic;date_time;regex;system;chrono;filesystem;thread;log"
        "timer;TIMER;system;chrono" "prg_exec_monitor;TEST;system;timer"
        "type_erasure;TYPE_ERASURE;system;chrono;thread"
        "unit_test_framework;TEST;system;timer"
        "wave;WAVE;date_time;system;chrono;filesystem;thread"
        "wserialization;SERIALIZATION;serialization")
      list(GET spec 0 lib)
      add_library("ext_boost_${lib}"
        "${EXT_BOOST_LIBRARY_TYPE}"
        IMPORTED
        GLOBAL)
      set_target_properties("ext_boost_${lib}"
        PROPERTIES
        IMPORTED_LOCATION
        "${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/lib/libboost_${lib}${EXT_BOOST_LIBRARY_SUFFIX}")
      list(LENGTH spec len)
      if(len GREATER_EQUAL 2)
        list(GET spec 1 LIB)
        target_compile_definitions("ext_boost_${lib}"
          INTERFACE
          "BOOST_${LIB}_DYN_LINK")
      endif()
      target_link_libraries("ext_boost_${lib}"
        INTERFACE
        ext_boost_headers)
      list(LENGTH spec len)
      if(len GREATER_EQUAL 3)
        list(REMOVE_AT spec 0 1)
        foreach(dep IN LISTS spec)
          target_link_libraries("ext_boost_${lib}"
            INTERFACE
            "ext_boost_${dep}")
        endforeach()
      endif()
      add_dependencies("ext_boost_${lib}"
        ext_boost)
    endforeach()

    if(NOT EXT_BOOST_SHARED_LIBS)
      add_library(ext_boost_exception
        STATIC
        IMPORTED
        GLOBAL)
      set_target_properties(ext_boost_exception
        PROPERTIES
        IMPORTED_LOCATION
        "${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/lib/libboost_exception${EXT_BOOST_LIBRARY_SUFFIX}")
      target_link_libraries(ext_boost_exception
        INTERFACE
        ext_boost_headers)
      add_dependencies(ext_boost_exception
        ext_boost)
    endif()

    add_library(ext_boost_stacktrace_addr2line
      "${EXT_BOOST_LIBRARY_TYPE}"
      IMPORTED
      GLOBAL)
    set_target_properties(ext_boost_stacktrace_addr2line
      PROPERTIES
      IMPORTED_LOCATION
      "${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/lib/libboost_stacktrace_addr2line${EXT_BOOST_LIBRARY_SUFFIX}")
    target_compile_definitions(ext_boost_stacktrace_addr2line
      INTERFACE
      BOOST_STACKTRACE_DYN_LINK
      BOOST_STACKTRACE_USE_ADDR2LINE)
    target_link_libraries(ext_boost_stacktrace_addr2line
      INTERFACE
      ext_boost_headers
      "-ldl"
      "-rdynamic")
    add_dependencies(ext_boost_stacktrace_addr2line
      ext_boost)

    add_library(ext_boost_stacktrace_backtrace
      "${EXT_BOOST_LIBRARY_TYPE}"
      IMPORTED
      GLOBAL)
    set_target_properties(ext_boost_stacktrace_backtrace
      PROPERTIES
      IMPORTED_LOCATION
      "${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/lib/libboost_stacktrace_backtrace${EXT_BOOST_LIBRARY_SUFFIX}")
    target_compile_definitions(ext_boost_stacktrace_backtrace
      INTERFACE
      BOOST_STACKTRACE_DYN_LINK
      BOOST_STACKTRACE_USE_BACKTRACE)
    target_link_libraries(ext_boost_stacktrace_backtrace
      INTERFACE
      ext_boost_headers
      "-lbacktrace"
      "-ldl"
      "-rdynamic")
    add_dependencies(ext_boost_stacktrace_backtrace
      ext_boost)

    add_library(ext_boost_stacktrace_basic
      "${EXT_BOOST_LIBRARY_TYPE}"
      IMPORTED
      GLOBAL)
    set_target_properties(ext_boost_stacktrace_basic
      PROPERTIES
      IMPORTED_LOCATION
      "${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/lib/libboost_stacktrace_basic${EXT_BOOST_LIBRARY_SUFFIX}")
    target_compile_definitions(ext_boost_stacktrace_basic
      INTERFACE
      BOOST_STACKTRACE_DYN_LINK)
    target_link_libraries(ext_boost_stacktrace_basic
      INTERFACE
      ext_boost_headers
      "-ldl"
      "-rdynamic")
    add_dependencies(ext_boost_stacktrace_basic
      ext_boost)

    add_library(ext_boost_stacktrace_noop
      "${EXT_BOOST_LIBRARY_TYPE}"
      IMPORTED
      GLOBAL)
    set_target_properties(ext_boost_stacktrace_noop
      PROPERTIES
      IMPORTED_LOCATION
      "${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/lib/libboost_stacktrace_noop${EXT_BOOST_LIBRARY_SUFFIX}")
    target_compile_definitions(ext_boost_stacktrace_noop
      INTERFACE
      BOOST_STACKTRACE_DYN_LINK
      BOOST_STACKTRACE_USE_NOOP)
    target_link_libraries(ext_boost_stacktrace_noop
      INTERFACE
      ext_boost_headers)
    add_dependencies(ext_boost_stacktrace_noop
      ext_boost)
  else()
    message(FATAL_ERROR
      "An unsupported system: CMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}.")
  endif()
else()
  add_library(ext_boost_headers
    INTERFACE)
  if(EXT_BOOST_PREFIX)
    set_target_include_directories(ext_boost_headers
      INTERFACE
      "${EXT_BOOST_PREFIX}/include")
  endif()

  if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang"
      OR CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    add_library(ext_boost_link_search_flags
      INTERFACE)
    if(EXT_ICU4C_PREFIX)
      target_link_libraries(ext_boost_link_search_flags
        "-L${EXT_ICU4C_PREFIX}/lib")
    endif()

    set(EXT_BOOST_LIBRARY_SUFFIX "")
    if(EXT_BOOST_LAYOUT STREQUAL "versioned")
      # TODO: Compiler version tag, e.g., `-gcc82'.
    endif()
    if(EXT_BOOST_LAYOUT STREQUAL "tagged"
        OR EXT_BOOST_LAYOUT STREQUAL "versioned")
      set(EXT_BOOST_LIBRARY_SUFFIX
        "${EXT_BOOST_LIBRARY_SUFFIX}-mt")
      # TODO: `s'
      if(EXT_BOOST_LIBRARY_SUFFIX STREQUAL "debug")
        set(EXT_BOOST_LIBRARY_SUFFIX
          "${EXT_BOOST_LIBRARY_SUFFIX}-g")
      endif()
    endif()
    if(EXT_BOOST_LAYOUT STREQUAL "versioned")
      # TODO: Architecture and address model tag, e.g., `-x64'.
      # TODO: Boost version tag, e.g., `-1_68'.
    endif()
    if(EXT_BOOST_BUILD_ID)
      set(EXT_BOOST_LIBRARY_SUFFIX
        "${EXT_BOOST_LIBRARY_SUFFIX}-${EXT_BOOST_BUILD_ID}")
    endif()

    foreach(spec "atomic;ATOMIC" "container;CONTAINER" "context;CONTEXT"
        "date_time;DATE_TIME" "exception" "iostreams;IOSTREAMS" "math_c99f"
        "math_c99l" "math_c99" "program_options;PROGRAM_OPTIONS" "regex;REGEX"
        "graph;GRAPH;regex" "serialization;SERIALIZATION" "signals;SIGNALS"
        "system;SYSTEM" "chrono;CHRONO;system" "contract;CONTRACT;system"
        "filesystem;FILESYSTEM;system"
        "fiber;FIBERS;context;system;filesystem" "random;RANDOM;system"
        "test_exec_monitor;TEST" "thread;THREAD;system"
        "coroutine;COROUTINES;context;system;chrono;thread"
        "locale;LOCALE;system;chrono;thread"
        "log;LOG;atomic;date_time;regex;system;chrono;filesystem;thread"
        "log_setup;LOG;atomic;date_time;regex;system;chrono;filesystem;thread;log"
        "timer;TIMER;system;chrono" "prg_exec_monitor;TEST;system;timer"
        "type_erasure;TYPE_ERASURE;system;chrono;thread"
        "unit_test_framework;TEST;system;timer"
        "wave;WAVE;date_time;system;chrono;filesystem;thread"
        "wserialization;SERIALIZATION;serialization")
      list(GET spec 0 lib)
      add_library("ext_boost_${lib}"
        INTERFACE)
      list(LENGTH spec len)
      if(len GREATER_EQUAL 2)
        list(GET spec 1 LIB)
        target_compile_definitions("ext_boost_${lib}"
          INTERFACE
          "BOOST_${LIB}_DYN_LINK")
      endif()
      target_link_libraries("ext_boost_${lib}"
        INTERFACE
        ext_boost_headers)
      if(len GREATER_EQUAL 3)
        list(REMOVE_AT spec 0 1)
        foreach(dep IN LISTS spec)
          target_link_libraries("ext_boost_${dep}"
            INTERFACE
            "ext_boost_${dep}")
        endforeach()
      endif()
      target_link_libraries("ext_boost_${lib}"
        INTERFACE
        ext_boost_link_search_flags
        "-lboost_${lib}${EXT_BOOST_LIBRARY_SUFFIX}")
    endforeach()

    add_library(ext_boost_stacktrace_addr2line
      INTERFACE)
    target_compile_definitions(ext_boost_stacktrace_addr2line
      INTERFACE
      BOOST_STACKTRACE_DYN_LINK
      BOOST_STACKTRACE_USE_ADDR2LINE)
    target_link_libraries(ext_boost_stacktrace_addr2line
      INTERFACE
      ext_boost_headers
      "-lboost_stacktrace_addr2line${EXT_BOOST_LIBRARY_SUFFIX}"
      -ldl
      -rdynamic)

    add_library(ext_boost_stacktrace_backtrace
      INTERFACE)
    target_compile_definitions(ext_boost_stacktrace_backtrace
      INTERFACE
      BOOST_STACKTRACE_DYN_LINK
      BOOST_STACKTRACE_USE_BACKTRACE)
    target_link_libraries(ext_boost_stacktrace_backtrace
      INTERFACE
      ext_boost_headers
      "-lboost_stacktrace_backtrace${EXT_BOOST_LIBRARY_SUFFIX}"
      -lbacktrace
      -ldl
      -rdynamic)

    add_library(ext_boost_stacktrace_basic
      INTERFACE)
    target_compile_definitions(ext_boost_stacktrace_basic
      INTERFACE
      BOOST_STACKTRACE_DYN_LINK)
    target_link_libraries(ext_boost_stacktrace_basic
      INTERFACE
      ext_boost_headers
      "-lboost_stacktrace_basic${EXT_BOOST_LIBRARY_SUFFIX}"
      -ldl
      -rdynamic)

    add_library(ext_boost_stacktrace_noop
      INTERFACE)
    target_compile_definitions(ext_boost_stacktrace_noop
      INTERFACE
      BOOST_STACKTRACE_DYN_LINK
      BOOST_STACKTRACE_USE_NOOP)
    target_link_libraries(ext_boost_stacktrace_noop
      INTERFACE
      ext_boost_headers
      "-lboost_stacktrace_noop${EXT_BOOST_LIBRARY_SUFFIX}")
  else()
    message(FATAL_ERROR
      "An unsupported compiler: CMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID}.")
  endif()
endif()
