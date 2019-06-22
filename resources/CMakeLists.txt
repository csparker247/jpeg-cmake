# CMake configuration for IJG libjpeg
# To install, copy this file and jconfig.h.in into a libjpeg source directory
# Adapted from LuaDist's CMakeLists
# https://github.com/LuaDist/libjpeg/blob/master/CMakeLists.txt

cmake_minimum_required(VERSION 3.0)

### Setup the project ###
file(READ "configure.ac" ac)
string(REGEX MATCH "AC_INIT\\(\\[libjpeg\\],\ \\[([0-9]*\\.[0-9]*\\.[0-9]*)\\]\\)" _ ${ac})
set(version ${CMAKE_MATCH_1})
project(libjpeg VERSION ${version} LANGUAGES C)
set(C_STANDARD 99)

### Options ###
include(CMakeDependentOption)
option(BUILD_SHARED_LIBS "Build shared libraries" ON)
option(BUILD_STATIC_LIBS "Build static libraries" ON)
cmake_dependent_option(BUILD_EXECUTABLES "Build JPEG utilities" ON "BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS" OFF)
# cmake_dependent_option(BUILD_TESTS "Build test executables" ON "BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS" OFF)
cmake_dependent_option(LINK_STATIC "Link all executables statically" OFF "BUILD_STATIC_LIBS;BUILD_EXECUTABLES" OFF)

# Make sure we build at least one library
if(NOT(BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS))
  message(FATAL_ERROR "Both static and shared libraries are disabled. Nothing will be built.")
endif()

### Configure jconfig.h ###
include(CheckIncludeFile)
check_include_file(stddef.h HAVE_STDDEF_H)
check_include_file(stdlib.h HAVE_STDLIB_H)
if(WIN32 AND NOT CYGWIN)
  set(TWO_FILE_COMMANDLINE ON)
endif()
configure_file(jconfig.h.in ${CMAKE_CURRENT_SOURCE_DIR}/jconfig.h)

### Build the object library ###
set(PUBLIC_HDRS
  jconfig.h
  jerror.h
  jmorecfg.h
  jpeglib.h
)

set(SRCS
  jaricom.c jcapimin.c jcapistd.c jcarith.c jccoefct.c jccolor.c
  jcdctmgr.c jchuff.c jcinit.c jcmainct.c jcmarker.c jcmaster.c jcomapi.c
  jcparam.c jcprepct.c jcsample.c jctrans.c jdapimin.c jdapistd.c jdarith.c
  jdatadst.c jdatasrc.c jdcoefct.c jdcolor.c jddctmgr.c jdhuff.c jdinput.c
  jdmainct.c jdmarker.c jdmaster.c jdmerge.c jdpostct.c jdsample.c jdtrans.c
  jerror.c jfdctflt.c jfdctfst.c jfdctint.c jidctflt.c jidctfst.c jidctint.c
  jmemmgr.c jmemnobs.c jquant1.c jquant2.c jutils.c
)

add_library(jpeg_objs OBJECT ${SRCS})
target_include_directories(jpeg_objs
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include>
)
set_target_properties(jpeg_objs PROPERTIES POSITION_INDEPENDENT_CODE ON)

### Create static and shared libs ###
if(BUILD_SHARED_LIBS)
  add_library(jpeg SHARED $<TARGET_OBJECTS:jpeg_objs>)
  set_target_properties(jpeg
    PROPERTIES
      VERSION ${PROJECT_VERSION_MAJOR}
      CLEAN_DIRECT_OUTPUT ON
      PUBLIC_HEADER "${PUBLIC_HDRS}"
  )
  install(TARGETS jpeg)
endif()

if(BUILD_STATIC_LIBS)
  add_library(jpeg_static STATIC $<TARGET_OBJECTS:jpeg_objs>)
    set_target_properties(jpeg_static
      PROPERTIES
        OUTPUT_NAME jpeg
        VERSION ${PROJECT_VERSION_MAJOR}
        CLEAN_DIRECT_OUTPUT ON
        PUBLIC_HEADER "${PUBLIC_HDRS}"
    )
    install(TARGETS jpeg_static)
endif()

# Configure and install pkg-config file
if(BUILD_STATIC_LIBS OR BUILD_SHARED_LIBS)
  include(GNUInstallDirs)
  # Handle the pc file's weird version number
  math(EXPR pc_vers_major "${PROJECT_VERSION_MAJOR} + ${PROJECT_VERSION_MINOR}")
  set(JPEG_LIB_VERSION "${pc_vers_major}:0:${PROJECT_VERSION_MINOR}")
  # Get the appropriate variables
  set(prefix ${CMAKE_INSTALL_PREFIX})
  set(libdir ${CMAKE_INSTALL_LIBDIR})
  set(includedir ${CMAKE_INSTALL_INCLUDEDIR})
  # Configure and install
  configure_file(libjpeg.pc.cmakein libjpeg.pc @ONLY)
  install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/libjpeg.pc
    DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/pkgconfig
  )
endif()

### Build executables
if(BUILD_EXECUTABLES)
  add_executable(cjpeg cdjpeg.c cjpeg.c rdbmp.c rdgif.c rdppm.c rdrle.c rdtarga.c rdswitch.c)
  add_executable(djpeg cdjpeg.c djpeg.c wrbmp.c wrgif.c wrppm.c wrrle.c wrtarga.c rdcolmap.c)
  add_executable(jpegtran jpegtran.c cdjpeg.c rdswitch.c transupp.c)
  add_executable(rdjpgcom rdjpgcom.c)
  add_executable(wrjpgcom wrjpgcom.c)

  # Link appropriately
  if(LINK_STATIC OR (NOT BUILD_SHARED_LIBS))
    message(STATUS "Linking executables statically")
    set(LINKED_JPEG_LIB jpeg_static)
  else()
    message(STATUS "Linking executables dynamically")
    set(LINKED_JPEG_LIB jpeg)
  endif()
  target_link_libraries(cjpeg ${LINKED_JPEG_LIB})
  target_link_libraries(djpeg ${LINKED_JPEG_LIB})
  target_link_libraries(jpegtran ${LINKED_JPEG_LIB})

  install(TARGETS cjpeg djpeg jpegtran rdjpgcom wrjpgcom)
endif()
