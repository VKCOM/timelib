cmake_minimum_required(VERSION 3.14..3.18)

# Build:
#   cmake .
#   make
#
# Install without cpack:
#
#   sudo make install
#
# Install with cpack (generates deb package):
#
#   cpack
#   sudo dpkg -i kphp-timelib_*.deb

# Included for CMAKE_INSTALL_PREFIX and other variables.
include(GNUInstallDirs)

project(kphp-timelib
        VERSION 1.0.0
        DESCRIPTION "timelib 2020.02 library fork for KPHP"
        HOMEPAGE_URL "https://github.com/derickr/timelib")

set(BASE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(OBJS_DIR "${BASE_DIR}/objs")

add_custom_target(kphp-timelib-codegen ALL DEPENDS parse_date.c parse_iso_intervals.c)
add_custom_command(
    OUTPUT   ${CMAKE_CURRENT_SOURCE_DIR}/parse_date.c
    COMMAND  re2c -d -b ${CMAKE_CURRENT_SOURCE_DIR}/parse_date.re > ${CMAKE_CURRENT_SOURCE_DIR}/parse_date.c
    DEPENDS  ${CMAKE_CURRENT_SOURCE_DIR}/parse_date.re
)
add_custom_command(
    OUTPUT   ${CMAKE_CURRENT_SOURCE_DIR}/parse_iso_intervals.c
    COMMAND  re2c -d -b ${CMAKE_CURRENT_SOURCE_DIR}/parse_iso_intervals.re > ${CMAKE_CURRENT_SOURCE_DIR}/parse_iso_intervals.c
    DEPENDS  ${CMAKE_CURRENT_SOURCE_DIR}/parse_iso_intervals.re
)

add_custom_target(kphp-timelib-includecopy ALL DEPENDS include/kphp/timelib/timelib.h)
add_custom_command(
    OUTPUT  ${CMAKE_CURRENT_SOURCE_DIR}/include/kphp/timelib/timelib.h
    COMMAND mkdir -p ${CMAKE_CURRENT_SOURCE_DIR}/include/kphp/timelib/ && cp ${CMAKE_CURRENT_SOURCE_DIR}/timelib.h ${CMAKE_CURRENT_SOURCE_DIR}/include/kphp/timelib/timelib.h
    DEPENDS timelib.h
)

set(TIMELIB_ALL_SOURCES
    astro.c
    docs
    dow.c
    interval.c
    parse_date.c
    parse_iso_intervals.c
    parse_tz.c
    parse_zoneinfo.c
    timelib.c
    tm2unixtime.c
    unixtime2tm.c)

set_source_files_properties(
    parse_date.c
    parse_iso_intervals.c
    PROPERTIES GENERATED TRUE)

add_library(kphp-timelib STATIC ${TIMELIB_ALL_SOURCES})
add_library(vk::kphp-timelib ALIAS kphp-timelib)

target_compile_options(kphp-timelib PUBLIC
    -std=c11
    -O3
    -Wall -Werror -Wextra -Wmaybe-uninitialized -Wmissing-field-initializers -Wshadow
    -Wno-unused-parameter
    -pedantic
    -DHAVE_STDINT_H -DHAVE_GETTIMEOFDAY -DHAVE_UNISTD_H -DHAVE_DIRENT_H
)
if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "7.0.0")
    message("GCC >= 7, adding extra compile options")
    target_compile_options(kphp-timelib PUBLIC -Wno-implicit-fallthrough)
  endif()
endif()

set_target_properties(kphp-timelib PROPERTIES ARCHIVE_OUTPUT_DIRECTORY ${OBJS_DIR})

target_link_libraries(kphp-timelib m)

set_target_properties(kphp-timelib PROPERTIES PUBLIC_HEADER "timelib.h")

add_dependencies(kphp-timelib kphp-timelib-includecopy kphp-timelib-codegen)

install(TARGETS kphp-timelib
        COMPONENT KPHP
        LIBRARY DESTINATION "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}"
        ARCHIVE DESTINATION "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}"
        PUBLIC_HEADER DESTINATION "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_INCLUDEDIR}/kphp/timelib")

set(CPACK_PACKAGING_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX})
set(CPACK_DEB_COMPONENT_INSTALL ON)
set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)
set(CPACK_COMPONENTS_GROUPING "IGNORE")
set(CPACK_GENERATOR "DEB")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "KPHP Team")
set(CPACK_DEBIAN_FILE_NAME "DEB-DEFAULT")
set(CPACK_PACKAGE_HOMEPAGE_URL ${CMAKE_PROJECT_HOMEPAGE_URL})
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "${CMAKE_PROJECT_DESCRIPTION}")

include(CPack)
