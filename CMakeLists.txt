# Copyright (C) 2014 MediaSift Ltd.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

#
# Project Properties
#
CMAKE_MINIMUM_REQUIRED (VERSION 3.2)

INCLUDE ("cmake/HunterGate.cmake")
HUNTERGATE (
    URL "https://github.com/ruslo/hunter/archive/v0.23.198.tar.gz"
    SHA1 "f44b698691eedfe9d1ee6763a6fb9b4af7756c24"
)

PROJECT (served VERSION 1.4.3)
SET (APPLICATION_NAME "Served HTTP REST Library")
SET (APPLICATION_CODENAME "${PROJECT_NAME}")
SET (APPLICATION_COPYRIGHT_YEARS "2014")
SET (APPLICATION_VERSION_MAJOR 1)
SET (APPLICATION_VERSION_MINOR 4)
SET (APPLICATION_VERSION_PATCH 3)
SET (APPLICATION_VERSION_TYPE DS1)
SET (APPLICATION_VERSION_STRING "${APPLICATION_VERSION_MAJOR}.${APPLICATION_VERSION_MINOR}.${APPLICATION_VERSION_PATCH}-${APPLICATION_VERSION_TYPE}")
SET (APPLICATION_VENDOR_ID "com.meltwater")
SET (APPLICATION_VENDOR_NAME "Meltwater")
SET (APPLICATION_VENDOR_URL "meltwater.com")
SET (APPLICATION_ID "${APPLICATION_VENDOR_ID}.${PROJECT_NAME}")

set(config_install_dir "lib/cmake/${PROJECT_NAME}")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

#
# Configurable Options
#
OPTION (SERVED_BUILD_TESTS "Build unit test suite" OFF)
OPTION (SERVED_BUILD_EXAMPLES "Build examples" OFF)
OPTION (SERVED_BUILD_RPM "Build RPM package" OFF)
OPTION (SERVED_BUILD_DEB "Build DEB package" OFF)

#
# Debugging Options
#
SET (CMAKE_VERBOSE_MAKEFILE 0) # Use 1 for debugging, 0 for release

#
# Locate Project Prerequisites
#
HUNTER_ADD_PACKAGE (Boost COMPONENTS system)
FIND_PACKAGE (Boost CONFIG COMPONENTS REQUIRED system)
set(Boost_LIBRARIES "Boost::system")

FIND_PACKAGE (Threads)

#
# Project Output Paths
#
SET (MAINFOLDER ${PROJECT_SOURCE_DIR})
SET (EXECUTABLE_OUTPUT_PATH "${MAINFOLDER}/bin")
SET (LIBRARY_OUTPUT_PATH "${MAINFOLDER}/lib")

#
# Project Search Paths
#
LIST (APPEND CMAKE_PREFIX_PATH "${MAINFOLDER}")
SET (CMAKE_MODULE_PATH "${MAINFOLDER}/cmake")
INCLUDE_DIRECTORIES ("${MAINFOLDER}/src")

INCLUDE (EnableStdCXX11)
ENABLE_STDCXX11 ()

#
# Enable warnings
#
IF (NOT MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pedantic -Wextra -Wno-missing-field-initializers")
ENDIF (NOT MSVC)

IF (MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D _WIN32_WINNT=0x0601 -D NOGDI")
ENDIF (MSVC)

#
# Configure Files
#
FILE (GLOB_RECURSE CONFIGINPUTS1 src/served/*.hpp.in.cmake)
LIST (APPEND CONFIGINPUTS "${CONFIGINPUTS1}")
FOREACH (CONFIGINPUT ${CONFIGINPUTS})
    STRING (REPLACE ".hpp.in.cmake" ".hpp" CONFIGOUTPUT "${CONFIGINPUT}")
    CONFIGURE_FILE ("${CONFIGINPUT}" "${CONFIGOUTPUT}")
ENDFOREACH (CONFIGINPUT ${CONFIGINPUTS})

#
# pkg-config file generator
#
CONFIGURE_FILE("install/pkgconfig/served.pc.in" "${LIBRARY_OUTPUT_PATH}/pkgconfig/served.pc" @ONLY)
INSTALL(FILES "${LIBRARY_OUTPUT_PATH}/pkgconfig/served.pc" DESTINATION lib/pkgconfig )

#
# Add Build Targets
#
IF (SERVED_BUILD_TESTS)
    ENABLE_TESTING (true)
ENDIF (SERVED_BUILD_TESTS)
ADD_SUBDIRECTORY (src)

target_include_directories(
    served
    PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}>"
)

include(CMakePackageConfigHelpers)

# Use:
#   * PROJECT_VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Use:
#   * TARGETS_EXPORT_NAME
#   * PROJECT_NAME
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

#
# CPack Generic
#
SET(CPACK_PACKAGE_NAME "libserved")
SET(CPACK_PACKAGE_VERSION "${APPLICATION_VERSION_MAJOR}.${APPLICATION_VERSION_MINOR}.${APPLICATION_VERSION_PATCH}")
SET(CPACK_PACKAGE_VERSION_MAJOR "${APPLICATION_VERSION_MAJOR}")
SET(CPACK_PACKAGE_VERSION_MINOR "${APPLICATION_VERSION_MINOR}")
SET(CPACK_PACKAGE_VERSION_PATCH "${APPLICATION_VERSION_PATCH}")
SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Served is a C++ library for building high performance RESTful web servers")
SET(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README.md")
SET(CPACK_PACKAGE_VENDOR "${APPLICATION_VENDOR_NAME}")
SET(CPACK_PACKAGE_URL "${APPLICATION_VENDOR_URL}")
SET(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.md")
SET(CPACK_SOURCE_GENERATOR "TGZ")

IF (SERVED_BUILD_DEB)
    INCLUDE(CPackConfigDEB)
ENDIF (SERVED_BUILD_DEB)
IF (SERVED_BUILD_RPM)
    INCLUDE(CPackConfigRPM)
ENDIF (SERVED_BUILD_RPM)

INCLUDE(CPack)
