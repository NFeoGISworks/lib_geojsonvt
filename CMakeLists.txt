################################################################################
# Project:  Lib JBIG
# Purpose:  CMake build scripts
# Author:   Dmitry Baryshnikov, dmitry.baryshnikov@nexgis.com
# Author:   NikitaFeodonit, nfeodonit@yandex.com
################################################################################
# Copyright (C) 2015, NextGIS <info@nextgis.com>
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included
# in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
# OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
# THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.
################################################################################

cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)

project(geojsonvtcpp CXX)

# some init settings
set(CMAKE_COLOR_MAKEFILE ON)
# set path to additional CMake modules
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

include(GNUInstallDirs)

set(INSTALL_BIN_DIR ${CMAKE_INSTALL_BINDIR} CACHE INTERNAL "Installation directory for executables" FORCE)
set(INSTALL_LIB_DIR ${CMAKE_INSTALL_LIBDIR} CACHE INTERNAL "Installation directory for libraries" FORCE)
set(INSTALL_INC_DIR ${CMAKE_INSTALL_INCLUDEDIR} CACHE INTERNAL "Installation directory for headers" FORCE)
set(INSTALL_DOC_DIR ${CMAKE_INSTALL_DOCDIR} CACHE INTERNAL "Share directory for data" FORCE)

set(INSTALL_INC_FULL_DIR "${CMAKE_INSTALL_PREFIX}/${INSTALL_INC_DIR}")

if(CMAKE_GENERATOR_TOOLSET MATCHES "v([0-9]+)_xp")
    add_definitions(-D_WIN32_WINNT=0x0501)
endif()

include(util)
check_version(GJVT_MAJOR_VERSION GJVT_MINOR_VERSION GJVT_REV_VERSION)
set(VERSION ${GJVT_MAJOR_VERSION}.${GJVT_MINOR_VERSION}.${GJVT_REV_VERSION})
report_version(${PROJECT_NAME} ${VERSION})


include(FindAnyProject)

find_anyproject(RapidJSON REQUIRED)
find_anyproject(VARIANT REQUIRED)

configure_file(${CMAKE_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake IMMEDIATE @ONLY)

set(PUBLIC_HEADER_DIR "${CMAKE_SOURCE_DIR}/include")

set(PUBLIC_HDRS
    include/mapbox/geojsonvt.hpp
    include/mapbox/geojsonvt/clip.hpp
    include/mapbox/geojsonvt/convert.hpp
    include/mapbox/geojsonvt/simplify.hpp
    include/mapbox/geojsonvt/tile.hpp
    include/mapbox/geojsonvt/transform.hpp
    include/mapbox/geojsonvt/types.hpp
    include/mapbox/geojsonvt/wrap.hpp
)

set(DOCS
    LICENSE
)

set(SRCS
    src/clip.cpp
    src/convert.cpp
    src/geojsonvt.cpp
    src/simplify.cpp
    src/tile.cpp
    src/transform.cpp
    src/wrap.cpp
)

include_directories ( ${CMAKE_SOURCE_DIR}/include )

set(LIB_NAME ${PROJECT_NAME})
if(BUILD_SHARED_LIBS)
    add_library(${LIB_NAME} SHARED ${SRCS})
    set_target_properties(${LIB_NAME} PROPERTIES SOVERSION 1)
else()
    add_library(${LIB_NAME} STATIC ${SRCS})
    set_target_properties(${LIB_NAME} PROPERTIES SOVERSION 1)
endif()

set_property(TARGET ${LIB_NAME} PROPERTY CXX_STANDARD 14)
set_property(TARGET ${LIB_NAME} PROPERTY CXX_STANDARD_REQUIRED ON)
target_include_directories(${LIB_NAME} PUBLIC "${INSTALL_INC_FULL_DIR}")
target_sources(${LIB_NAME} PUBLIC ${PUBLIC_HDRS})
target_link_extlibraries(${LIB_NAME})

export(TARGETS ${LIB_NAME} FILE ${LIB_NAME}-exports.cmake EXPORT_LINK_INTERFACE_LIBRARIES)

add_custom_target(uninstall COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)


if(NOT CYGWIN)
    set_target_properties(${LIB_NAME} PROPERTIES VERSION ${VERSION})
endif()

if(UNIX)
    # On unix-like platforms the library is almost always called lib${PROJECT_NAME}
    set_target_properties(${LIB_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME})
endif()

if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL)
    install(TARGETS ${LIB_NAME}
        EXPORT ${LIB_NAME}
        RUNTIME DESTINATION "${INSTALL_BIN_DIR}"
        ARCHIVE DESTINATION "${INSTALL_LIB_DIR}"
        LIBRARY DESTINATION "${INSTALL_LIB_DIR}" )
endif()

if(NOT SKIP_INSTALL_HEADERS AND NOT SKIP_INSTALL_ALL)
    install(DIRECTORY ${PUBLIC_HEADER_DIR}/ DESTINATION "${INSTALL_INC_DIR}")
    install(FILES ${DOCS} DESTINATION "${INSTALL_DOC_DIR}/lib${PROJECT_NAME}")
endif()
