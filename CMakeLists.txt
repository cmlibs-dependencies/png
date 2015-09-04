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

# COMMAND LINE OPTIONS
option(PNG_SHARED "Build shared lib" YES)
option(PNG_STATIC "Build static lib" YES)

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
set(PNG_LIB_NAME png${PNGLIB_MAJOR}${PNGLIB_MINOR})

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

if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux" AND ${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64" )
	SET_SOURCE_FILES_PROPERTIES(${libpng_sources}
		PROPERTIES COMPILE_FLAGS "-fPIC")
endif(${CMAKE_SYSTEM_NAME} STREQUAL "Linux" AND ${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64" )

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

#if(NOT M_LIBRARY AND NOT WIN32)
#  add_definitions(-DPNG_NO_FLOATING_POINT_SUPPORTED)
#endif()

# NOW BUILD OUR TARGET
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

set(PNG_LIBRARIES)
if(PNG_SHARED)
  add_library(${PNG_LIB_NAME} SHARED ${libpng_sources})
  set(PNG_LIBRARIES ${PNG_LIB_NAME})
  set_target_properties(${PNG_LIB_NAME} PROPERTIES DEFINE_SYMBOL ZLIB_DLL)
  if(MSVC)
    # msvc does not append 'lib' - do it here to have consistent name
    set_target_properties(${PNG_LIB_NAME} PROPERTIES PREFIX "lib")
    set_target_properties(${PNG_LIB_NAME} PROPERTIES IMPORT_PREFIX "lib")
  endif()
  if(WIN32)
    set_target_properties(${PNG_LIB_NAME} PROPERTIES DEFINE_SYMBOL PNG_BUILD_DLL)
  endif()
  target_link_libraries(${PNG_LIB_NAME} zlib)
  if (UNIX)
      target_link_libraries(${PNG_LIB_NAME} m)
  endif()
endif()

if(PNG_STATIC)
  set(PNG_LIB_NAME_STATIC ${PNG_LIB_NAME}_static)
  add_library(${PNG_LIB_NAME_STATIC} STATIC ${libpng_sources})
  set(PNG_LIBRARIES "${PNG_LIBRARIES} ${PNG_LIB_NAME_STATIC}")
  if(MSVC)
    # msvc does not append 'lib' - do it here to have consistent name
    set_target_properties(${PNG_LIB_NAME_STATIC} PROPERTIES PREFIX "lib")
  else()
    set_target_properties(${PNG_LIB_NAME_STATIC} PROPERTIES DEFINE_SYMBOL ZLIB_DLL)
  endif()
  target_link_libraries(${PNG_LIB_NAME_STATIC} zlib)
  if (UNIX)
      target_link_libraries(${PNG_LIB_NAME_STATIC} m)
  endif()
endif()

enable_testing()
if(PNG_TESTS AND PNG_SHARED)
  # does not work with msvc due to png_lib_ver issue
  add_executable(pngtest ${pngtest_sources})
  target_link_libraries(pngtest ${PNG_LIB_NAME})
  add_test(pngtest ./pngtest ${CMAKE_CURRENT_SOURCE_DIR}/pngtest.png)
endif()

#
# Set a variable with CMake code which:
# Creates a symlink from src to dest (if possible) or alternatively
# copies if different.
macro(_png_generate_symlink_code CODE SRC DEST)
  if(WIN32 AND NOT CYGWIN)
    set(_png_gsc_message "Copying ${SRC} to ${DEST} if needed")
    set(_png_gsc_operation "copy_if_different")
  else()
    set(_png_gsc_message "Symlinking ${SRC} to ${DEST}")
    set(_png_gsc_operation "create_symlink")
  endif()

  set(${CODE} "
    message(STATUS \"${_png_gsc_message}\")
    execute_process(COMMAND \${CMAKE_COMMAND} -E ${_png_gsc_operation}
      \"${SRC}\" \"${DEST}\")
    ")
endmacro()

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
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/libpng.pc.in
  ${CMAKE_CURRENT_BINARY_DIR}/${PNGLIB_NAME}.pc @ONLY)
_png_generate_symlink_code(PNG_PC_INSTALL_CODE
  ${CMAKE_CURRENT_BINARY_DIR}/${PNGLIB_NAME}.pc
  ${CMAKE_CURRENT_BINARY_DIR}/libpng.pc)
install(CODE ${PNG_PC_INSTALL_CODE})

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/libpng-config.in
  ${CMAKE_CURRENT_BINARY_DIR}/${PNGLIB_NAME}-config @ONLY)
_png_generate_symlink_code(PNG_CONFIG_INSTALL_CODE
  ${CMAKE_CURRENT_BINARY_DIR}/${PNGLIB_NAME}-config
  ${CMAKE_CURRENT_BINARY_DIR}/libpng-config)
install(CODE ${PNG_CONFIG_INSTALL_CODE})

# SET UP LINKS
if(PNG_SHARED)
  set_target_properties(${PNG_LIB_NAME} PROPERTIES
    SOVERSION 15
    CLEAN_DIRECT_OUTPUT 1)
endif()
if(PNG_STATIC)
  # MSVC doesn't use a different file extension for shared vs. static
  # libs.  We are able to change OUTPUT_NAME to remove the _static
  # for all other platforms.
  if(NOT MSVC)
    set_target_properties(${PNG_LIB_NAME_STATIC} PROPERTIES
      OUTPUT_NAME ${PNG_LIB_NAME}
      CLEAN_DIRECT_OUTPUT 1)
  endif()
endif()

# INSTALL
if(PNG_SHARED)
    install(TARGETS ${PNG_LIB_NAME}
        EXPORT png-config
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
    
    # Create a symlink for libpng.dll.a => libpng15.dll.a on Cygwin
    if(CYGWIN)
      _png_generate_symlink_code(PNG_SHARED_IMPLIB_INSTALL_CODE
        ${PNGLIB_NAME}${CMAKE_IMPORT_LIBRARY_SUFFIX}
        libpng${CMAKE_IMPORT_LIBRARY_SUFFIX})
      install(CODE ${PNG_SHARED_IMPLIB_INSTALL_CODE})
    endif()
    
    if(NOT WIN32)
      IF(CMAKE_LIBRARY_OUTPUT_DIRECTORY)
        _png_generate_symlink_code(PNG_SHARED_INSTALL_CODE
             ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/${PNGLIB_NAME}${CMAKE_SHARED_LIBRARY_SUFFIX}
             ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/libpng${CMAKE_SHARED_LIBRARY_SUFFIX})
                install(CODE ${PNG_SHARED_INSTALL_CODE})
      ELSE(CMAKE_LIBRARY_OUTPUT_DIRECTORY)
        _png_generate_symlink_code(PNG_SHARED_INSTALL_CODE
           ${PNGLIB_NAME}${CMAKE_SHARED_LIBRARY_SUFFIX}
           libpng${CMAKE_SHARED_LIBRARY_SUFFIX})
        install(CODE ${PNG_SHARED_INSTALL_CODE})
      ENDIF(CMAKE_LIBRARY_OUTPUT_DIRECTORY)
    endif()
endif()

if(PNG_STATIC)
    install(TARGETS ${PNG_LIB_NAME_STATIC}
        EXPORT png-config
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
    if(NOT WIN32 OR CYGWIN)
      IF(CMAKE_ARCHIVE_OUTPUT_DIRECTORY)
         _png_generate_symlink_code(PNG_STATIC_INSTALL_CODE
            ${CMAKE_ARCHIVE_OUTPUT_DIRECTORY}/${PNGLIB_NAME}${CMAKE_STATIC_LIBRARY_SUFFIX}
        ${CMAKE_ARCHIVE_OUTPUT_DIRECTORY}/libpng${CMAKE_STATIC_LIBRARY_SUFFIX})
        install(CODE ${PNG_STATIC_INSTALL_CODE})
      ELSE(CMAKE_ARCHIVE_OUTPUT_DIRECTORY)
        _png_generate_symlink_code(PNG_STATIC_INSTALL_CODE
           ${PNGLIB_NAME}${CMAKE_STATIC_LIBRARY_SUFFIX}
           libpng${CMAKE_STATIC_LIBRARY_SUFFIX})
        install(CODE ${PNG_STATIC_INSTALL_CODE})
      ENDIF(CMAKE_ARCHIVE_OUTPUT_DIRECTORY)
    endif()
endif()

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
#file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/hypre-config-dependencies.cmake "find_dependency(LAPACK ${LAPACK_VERSION})\r\n")
#if (WITH_OPENMP)
#    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/hypre-config-dependencies.cmake "find_dependency(OpenMP)\r\n")
#endif()
#if (WITH_SUPERLU AND NOT SUPERLU_INTERNAL)
#    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/hypre-config-dependencies.cmake "find_dependency(SUPERLU)\r\n")
#endif()
file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/png-config-dependencies.cmake "set(_IMPORT_PREFIX \${${PROJECT_NAME}_IMPORT_PREFIX})")
install(FILES 
    ${CMAKE_CURRENT_BINARY_DIR}/png-config-version.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/png-config-dependencies.cmake
    DESTINATION ${PACKAGE_CONFIG_DIR})	
