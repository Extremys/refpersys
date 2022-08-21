cmake_minimum_required(VERSION 3.2.3)
project(RefPerfSys)

set(CMAKE_BUILD_TYPE Debug)
set(CMAKE_CXX_STANDARD 17)
set(EXE_NAME refpersys)

#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -stdlib=libc++ -Wall -Werror")

find_package(Git)
execute_process(COMMAND
  "${GIT_EXECUTABLE}" describe --match=NeVeRmAtCh --always --abbrev=40 --dirty
  WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
  OUTPUT_VARIABLE RPS_GITID
  ERROR_QUIET OUTPUT_STRIP_TRAILING_WHITESPACE)
add_definitions( -DRPS_GITID="${RPS_GITID}" )

set(FLTK_DIR /Users/valentin/dev/fltk/install/FLTK.framework/Resources/CMake/)
find_package(FLTK REQUIRED NO_MODULE)
#include_directories(${FLTK_INCLUDE_DIRS})

set(LIBARGP_DIR /opt/homebrew/Cellar/argp-standalone/1.3/)
include_directories(${LIBARGP_DIR}/include)
link_directories(${LIBARGP_DIR}/lib)

set(LIBUNISTRING_DIR /opt/homebrew/Cellar/libunistring/1.0)
include_directories(${LIBUNISTRING_DIR}/include)
link_directories(${LIBUNISTRING_DIR}/lib)

set(LIBBACKTRACE_DIR /Users/valentin/dev/libbacktrace/install/)
include_directories(${LIBBACKTRACE_DIR}/include)
link_directories(${LIBBACKTRACE_DIR}/lib)

find_package(PkgConfig REQUIRED)
pkg_check_modules(JSONCPP REQUIRED jsoncpp)
#/opt/homebrew/Cellar/jsoncpp/1.9.5/lib/
include_directories(${JSONCPP_INCLUDE_DIRS})
link_directories(${JSONCPP_LIBRARY_DIRS})



set(CMAKE_THREAD_PREFER_PTHREAD TRUE) # these are to use the prefered -pthread flag
set(THREADS_PREFER_PTHREAD_FLAG TRUE) # over the -lpthread option
find_package(Threads REQUIRED)

file(GLOB RPS_CORE_HEADERS  *_rps.hh)
file(GLOB RPS_CORE_SOURCES  agenda_rps.cc
                            backtrace_rps.cc
                            cmdrepl_rps.cc
                            curl_rps.cc
                            dump_rps.cc
                            garbcoll_rps.cc
                            lexer_rps.cc
                            load_rps.cc
                            magicattrs_rps.cc
                            morevalues_rps.cc
                            objects_rps.cc
                            parsrepl_rps.cc
                            primes_rps.cc
                            repl_rps.cc
                            scalar_rps.cc
                            strbufdict_rps.cc
                            values_rps.cc )
file(GLOB RPS_JSONRPC_SOURCES *jsonrpc*_rps.cc)
file(GLOB RPS_BISON_SOURCES   *_rps.yy)

list(SORT RPS_CORE_HEADERS)
#message("${RPS_CORE_HEADERS}")
#message("${RPS_CORE_SOURCES}")
#message("${RPS_JSONRPC_SOURCES}")
#message("${RPS_BISON_SOURCES}")

if (APPLE)
  add_executable(${EXE_NAME} MACOSX_BUNDLE ${RPS_CORE_HEADERS} ${RPS_CORE_SOURCES} ${RPS_JSONRPC_SOURCES} ${RPS_BISON_SOURCES})
  target_link_libraries(${EXE_NAME} "-framework cocoa")
else ()
  add_executable(${EXE_NAME} ${RPS_CORE_HEADERS} ${RPS_CORE_SOURCES} ${RPS_JSONRPC_SOURCES} ${RPS_BISON_SOURCES})
endif (APPLE)


target_compile_options(${EXE_NAME} PUBLIC ${JSONCPP_CFLAGS_OTHER})
target_include_directories(${EXE_NAME} PUBLIC ${FLTK_INCLUDE_DIRS} ${JSONCPP_INCLUDE_DIRS} ${LIBARGP_DIR}/include ${LIBUNISTRING_DIR}/include ${LIBUNISTRING_DIR}/include)
target_link_libraries(${EXE_NAME} fltk Threads::Threads jsoncpp argp unistring backtrace)


