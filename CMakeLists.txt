# Copyright (C) 2007-2011 Glenn Randers-Pehrson

# This code is released under the libpng license.
# For conditions of distribution and use, see the disclaimer
# and license in png.h
cmake_minimum_required(VERSION 3.0)
project(PNG VERSION 1.5.2 LANGUAGES C)

option(BUILD_TESTS "${PROJECT_NAME} - Build tests" ON)
set(PACKAGE_CONFIG_DIR "lib/cmake" CACHE STRING "Directory for package config files (relative to CMAKE_INSTALL_PREFIX)")

# needed packages
find_package(ZLIB ${ZLIB_VERSION} REQUIRED)

option(PNG_NO_CONSOLE_IO "FIXME" YES)
option(PNG_NO_STDIO      "FIXME" YES)
option(PNG_DEBUG         "Build with debug output" NO)
option(PNGARG            "FIXME" YES)

# Use the prebuilt pnglibconf.h file from the scripts folder
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/scripts/pnglibconf.h.prebuilt
               ${CMAKE_CURRENT_BINARY_DIR}/pnglibconf.h COPYONLY)
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
if (ENABLE_PIC)
    set_target_properties(png PROPERTIES POSITION_INDEPENDENT_CODE ON)
endif()
target_link_libraries(png z)
if (UNIX)
    target_link_libraries(png m)
endif()
target_compile_definitions(png PRIVATE PNG_CONFIGURE_LIBPNG)
if(_AIX)
  target_compile_definitions(png PRIVATE _ALL_SOURCE)
endif()
if(MSVC)
  target_compile_definitions(png PRIVATE PNG_NO_MODULEDEF _CRT_SECURE_NO_DEPRECATE)
endif()
if(PNG_CONSOLE_IO_SUPPORTED)
  target_compile_definitions(png PRIVATE PNG_CONSOLE_IO_SUPPORTED)
endif()
if(PNG_NO_CONSOLE_IO)
  target_compile_definitions(png PRIVATE PNG_NO_CONSOLE_IO)
endif()
if(PNG_NO_STDIO)
  target_compile_definitions(png PRIVATE PNG_NO_STDIO)
endif()
if(PNG_DEBUG)
  target_compile_definitions(png PRIVATE PNG_DEBUG)
endif()
if(MSVC)
  target_compile_definitions(png PRIVATE PNG_NO_FLOATING_POINT_SUPPORTED)
endif()

if(BUILD_TESTS)#AND BUILD_SHARED_LIBS
  enable_testing()
  # does not work with msvc due to png_lib_ver issue
  add_executable(runpngtest ${pngtest_sources})
  target_link_libraries(runpngtest png)
  add_test(pngtest runpngtest ${CMAKE_CURRENT_SOURCE_DIR}/pngtest.png)
endif()

# INSTALL
install(TARGETS png
        EXPORT png-config
        DESTINATION lib
        INCLUDES DESTINATION include)

install(FILES ${libpng_public_hdrs}
    DESTINATION include)
install(EXPORT png-config
    DESTINATION ${PACKAGE_CONFIG_DIR})
include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(${CMAKE_CURRENT_BINARY_DIR}/png-config-version.cmake
    COMPATIBILITY AnyNewerVersion)
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/png-config-dependencies.cmake 
    "include(CMakeFindDependencyMacro)\r\n"
	"set(${PROJECT_NAME}_IMPORT_PREFIX \${_IMPORT_PREFIX})\r\n"
)
file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/png-config-dependencies.cmake "set(ZLIB_FIND_SYSTEM ${ZLIB_FIND_SYSTEM})\r\nfind_dependency(ZLIB)\r\n")
file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/png-config-dependencies.cmake "set(_IMPORT_PREFIX \${${PROJECT_NAME}_IMPORT_PREFIX})")
install(FILES 
    ${CMAKE_CURRENT_BINARY_DIR}/png-config-version.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/png-config-dependencies.cmake
    DESTINATION ${PACKAGE_CONFIG_DIR})	
