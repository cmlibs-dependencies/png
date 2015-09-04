cmake_minimum_required(VERSION 3.0)

project(PNG VERSION 1.5.2 LANGUAGES C)
# Copyright (C) 2007-2011 Glenn Randers-Pehrson

# This code is released under the libpng license.
# For conditions of distribution and use, see the disclaimer
# and license in png.h

set(PNGLIB_MAJOR 1)
set(PNGLIB_MINOR 5)
set(PNGLIB_RELEASE 2)
set(PNGLIB_NAME libpng${PNGLIB_MAJOR}${PNGLIB_MINOR})
set(PNGLIB_VERSION ${PNGLIB_MAJOR}.${PNGLIB_MINOR}.${PNGLIB_RELEASE})

# needed packages
find_package(ZLIB ${ZLIB_VERSION} REQUIRED)

if(MINGW)
  option(PNG_TESTS  "Build pngtest" NO)
else()
  option(PNG_TESTS  "Build pngtest" YES)
endif()

option(PNG_NO_CONSOLE_IO "FIXME" YES)
option(PNG_NO_STDIO      "FIXME" YES)
option(PNG_DEBUG         "Build with debug output" NO)
option(PNGARG            "FIXME" YES)

# SET LIBNAME
set(PNG_LIB_NAME png)

# to distinguish between debug and release lib
set(CMAKE_DEBUG_POSTFIX "d")

# Use the prebuilt pnglibconf.h file from the scripts folder
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/scripts/pnglibconf.h.prebuilt
               ${CMAKE_CURRENT_BINARY_DIR}/pnglibconf.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

# OUR SOURCES
set(libpng_public_hdrs
  png.h
  pngconf.h
  ${CMAKE_CURRENT_BINARY_DIR}/pnglibconf.h
)
set(libpng_sources
  ${libpng_public_hdrs}
  pngdebug.h
  pnginfo.h
  pngpriv.h
  pngstruct.h
  png.c
  pngerror.c
  pngget.c
  pngmem.c
  pngpread.c
  pngread.c
  pngrio.c
  pngrtran.c
  pngrutil.c
  pngset.c
  pngtrans.c
  pngwio.c
  pngwrite.c
  pngwtran.c
  pngwutil.c
)
set(pngtest_sources
  pngtest.c
)
# SOME NEEDED DEFINITIONS
add_definitions(-DPNG_CONFIGURE_LIBPNG)

if(_AIX)
  add_definitions(-D_ALL_SOURCE)
endif(_AIX)

if(MSVC)
  add_definitions(-DPNG_NO_MODULEDEF -D_CRT_SECURE_NO_DEPRECATE)
endif(MSVC)

if(PNG_CONSOLE_IO_SUPPORTED)
  add_definitions(-DPNG_CONSOLE_IO_SUPPORTED)
endif()

if(PNG_NO_CONSOLE_IO)
  add_definitions(-DPNG_NO_CONSOLE_IO)
endif()

if(PNG_NO_STDIO)
  add_definitions(-DPNG_NO_STDIO)
endif()

if(PNG_DEBUG)
  add_definitions(-DPNG_DEBUG)
endif()

if(MSVC)
  add_definitions(-DPNG_NO_FLOATING_POINT_SUPPORTED)
endif()

add_library(png ${libpng_sources})
set_target_properties(png PROPERTIES DEFINE_SYMBOL ZLIB_DLL)
if(MSVC)
    # msvc does not append 'lib' - do it here to have consistent name
    set_target_properties(png PROPERTIES PREFIX "lib")
    set_target_properties(png PROPERTIES IMPORT_PREFIX "lib")
endif()
if(WIN32)
    set_target_properties(png PROPERTIES DEFINE_SYMBOL PNG_BUILD_DLL)
endif()
target_link_libraries(png zlib)
if (UNIX)
    target_link_libraries(png m)
endif()

enable_testing()
if(PNG_TESTS AND PNG_SHARED)
  # does not work with msvc due to png_lib_ver issue
  add_executable(runpngtest ${pngtest_sources})
  target_link_libraries(runpngtest png)
  add_test(pngtest runpngtest ${CMAKE_CURRENT_SOURCE_DIR}/pngtest.png)
endif()

#
# Set a variable with CMake code which:
# Creates a symlink from src to dest (if possible) or alternatively
# copies if different.
#macro(_png_generate_symlink_code CODE SRC DEST)
#  if(WIN32 AND NOT CYGWIN)
#    set(_png_gsc_message "Copying ${SRC} to ${DEST} if needed")
#    set(_png_gsc_operation "copy_if_different")
#  else()
#    set(_png_gsc_message "Symlinking ${SRC} to ${DEST}")
#    set(_png_gsc_operation "create_symlink")
#  endif()
#
#  set(${CODE} "
#    message(STATUS \"${_png_gsc_message}\")
#    execute_process(COMMAND \${CMAKE_COMMAND} -E ${_png_gsc_operation}
#      \"${SRC}\" \"${DEST}\")
#    ")
#endmacro()

# CREATE PKGCONFIG FILES
# we use the same files like ./configure, so we have to set its vars
if(NOT DEFINED CMAKE_INSTALL_LIBDIR)
  set(CMAKE_INSTALL_LIBDIR lib)
endif(NOT DEFINED CMAKE_INSTALL_LIBDIR)
set(prefix      ${CMAKE_INSTALL_PREFIX})
set(exec_prefix ${CMAKE_INSTALL_PREFIX})
set(libdir      ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR})
set(includedir  ${CMAKE_INSTALL_PREFIX}/include)
set(LIBS        "-lz -lm")
#configure_file(${CMAKE_CURRENT_SOURCE_DIR}/libpng.pc.in
#  ${CMAKE_CURRENT_BINARY_DIR}/${PNGLIB_NAME}.pc @ONLY)
#_png_generate_symlink_code(PNG_PC_INSTALL_CODE
#  ${CMAKE_CURRENT_BINARY_DIR}/${PNGLIB_NAME}.pc
#  ${CMAKE_CURRENT_BINARY_DIR}/libpng.pc)
#install(CODE ${PNG_PC_INSTALL_CODE})

#configure_file(${CMAKE_CURRENT_SOURCE_DIR}/libpng-config.in
#  ${CMAKE_CURRENT_BINARY_DIR}/${PNGLIB_NAME}-config @ONLY)
#_png_generate_symlink_code(PNG_CONFIG_INSTALL_CODE
#  ${CMAKE_CURRENT_BINARY_DIR}/${PNGLIB_NAME}-config
#  ${CMAKE_CURRENT_BINARY_DIR}/libpng-config)
#install(CODE ${PNG_CONFIG_INSTALL_CODE})

# SET UP LINKS
if(BUILD_SHARED_LIBS)
  set_target_properties(png PROPERTIES
    SOVERSION 15
    CLEAN_DIRECT_OUTPUT 1)
else()
  # MSVC doesn't use a different file extension for shared vs. static
  # libs.  We are able to change OUTPUT_NAME to remove the _static
  # for all other platforms.
  if(NOT MSVC)
    set_target_properties(${PNG_LIB_NAME_STATIC} PROPERTIES
      OUTPUT_NAME png
      CLEAN_DIRECT_OUTPUT 1)
  endif()
endif()

# INSTALL
install(TARGETS png
        EXPORT png-config
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(FILES ${libpng_public_hdrs}
    DESTINATION include)
install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/libpng-config
    DESTINATION bin)
install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/${PNGLIB_NAME}-config
    DESTINATION bin)
install(EXPORT png-config
    DESTINATION ${PACKAGE_CONFIG_DIR})
include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(${CMAKE_CURRENT_BINARY_DIR}/png-config-version.cmake
    COMPATIBILITY AnyNewerVersion)
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/png-config-dependencies.cmake 
    "include(CMakeFindDependencyMacro)\r\n"
	"set(${PROJECT_NAME}_IMPORT_PREFIX \${_IMPORT_PREFIX})\r\n"
)
file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/png-config-dependencies.cmake "find_dependency(ZLIB)\r\n")
file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/png-config-dependencies.cmake "set(_IMPORT_PREFIX \${${PROJECT_NAME}_IMPORT_PREFIX})")
install(FILES 
    ${CMAKE_CURRENT_BINARY_DIR}/png-config-version.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/png-config-dependencies.cmake
    DESTINATION ${PACKAGE_CONFIG_DIR})	
