cmake_minimum_required(VERSION 3.20)

project(osgEarthBuilder)

option(DOWNLOAD_ONLY "Enabled download-only online mode" OFF)

if ("${CMAKE_BUILD_TYPE}" STREQUAL "")
    set(CMAKE_BUILD_TYPE "Debug")
endif()

# Add the install prefix to CMAKE_PREFIX_PATH
list(APPEND CMAKE_PREFIX_PATH "${CMAKE_INSTALL_PREFIX}")

include(ExternalProject)

set(DOWNLOAD_DIRECTORY "${CMAKE_SOURCE_DIR}/dist")

# Versions
set(LIBZLIB_VERSION "1.2.12")
set(LIBZLIB_URL "https://github.com/madler/zlib/archive/refs/tags/v${LIBZLIB_VERSION}.zip")
set(LIBZLIB_PACKAGE "libzlib-v${LIBZLIB_VERSION}.zip")

set(LIBTIFF_VERSION "4.3.0")
set(LIBTIFF_URL "https://gitlab.com/libtiff/libtiff/-/archive/v${LIBTIFF_VERSION}/libtiff-v${LIBTIFF_VERSION}.zip")
set(LIBTIFF_PACKAGE "libtiff-v${LIBTIFF_VERSION}.zip")

set(LIBPNG_VERSION "1.6.35")
set(LIBPNG_URL "https://github.com/glennrp/libpng/archive/refs/tags/v${LIBPNG_VERSION}.zip")
set(LIBPNG_PACKAGE "libpng-v${LIBPNG_VERSION}.zip")

set(SQLITE3_VERSION "3.38.2")
set(SQLITE3_URL "https://github.com/azadkuh/sqlite-amalgamation")  # this is a GIT repository since no prebuild source packages are available.
set(SQLITE3_PACKAGE "sqlite-v${SQLITE3_VERSION}.zip")

set(LIBCURL_VERSION "7.82.0")
string(REPLACE "." "_" LIBCURL_VERSION_UNDERSCORE ${LIBCURL_VERSION})
set(LIBCURL_URL "https://github.com/curl/curl/releases/download/curl-${LIBCURL_VERSION_UNDERSCORE}/curl-${LIBCURL_VERSION}.zip")
set(LIBCURL_PACKAGE "libcurl-v${LIBCURL_VERSION}.zip")

#
# Qt 5
#
# ExternalProject_Add(
#     qt5
#     GIT_REPOSITORY https://github.com/qt/qt5.git
#     GIT_TAG        v5.13.1
#     CMAKE_ARGS
#         -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
# )

#
# libzlib (OSG dependency)
#
if(DOWNLOAD_ONLY)
    ExternalProject_Add(
        libzlib
        URL "${LIBZLIB_URL}"
        DOWNLOAD_NAME "${LIBZLIB_PACKAGE}"
        DOWNLOAD_DIR "${DOWNLOAD_DIRECTORY}"
        DOWNLOAD_NO_EXTRACT ON
        DOWNLOAD_NO_PROGRESS ON
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
    )
else()
    ExternalProject_Add(
        libzlib
        URL "${DOWNLOAD_DIRECTORY}/${LIBZLIB_PACKAGE}"
        CMAKE_ARGS
            -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
            -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    )
endif()

#
# libTIFF (PROJ/OSG dependency)
#
if(DOWNLOAD_ONLY)
    ExternalProject_Add(
        libtiff
        URL "${LIBTIFF_URL}"
        DOWNLOAD_NAME "${LIBTIFF_PACKAGE}"
        DOWNLOAD_DIR "${DOWNLOAD_DIRECTORY}"
        DOWNLOAD_NO_EXTRACT ON
        DOWNLOAD_NO_PROGRESS ON
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
    )
else()
    ExternalProject_Add(
        libtiff
        URL "${DOWNLOAD_DIRECTORY}/${LIBTIFF_PACKAGE}"
        DEPENDS        libzlib
        CMAKE_ARGS
            -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
            -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
            -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    )
endif()

#
# libPNG (OSG dependency)
#
if(DOWNLOAD_ONLY)
    ExternalProject_Add(
        libpng
        URL "${LIBPNG_URL}"
        DOWNLOAD_NAME "${LIBPNG_PACKAGE}"
        DOWNLOAD_DIR "${DOWNLOAD_DIRECTORY}"
        DOWNLOAD_NO_EXTRACT ON
        DOWNLOAD_NO_PROGRESS ON
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
    )
else()
    ExternalProject_Add(
        libpng
        URL "${DOWNLOAD_DIRECTORY}/${LIBPNG_PACKAGE}"
        DEPENDS        libzlib
        CMAKE_ARGS
            -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
            -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
            -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    )
endif()

#
# SQLite (PROJ dependency)
#
if(DOWNLOAD_ONLY)
    ExternalProject_Add(
        sqlite
        GIT_REPOSITORY "${SQLITE3_URL}"
        GIT_TAG        ${SQLITE3_VERSION}
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
    )

    ExternalProject_Get_Property(sqlite SOURCE_DIR)

    cmake_path(REPLACE_FILENAME SOURCE_DIR "sqlite-v${SQLITE3_VERSION}" OUTPUT_VARIABLE RENAMED_SOURCE_DIR)
    
    add_custom_target(zip ALL
        COMMAND "${CMAKE_COMMAND}" -E rename "${SOURCE_DIR}" "${RENAMED_SOURCE_DIR}"
        COMMAND "${CMAKE_COMMAND}" -E tar "cfv" "${DOWNLOAD_DIRECTORY}/${SQLITE3_PACKAGE}" --format=zip "${RENAMED_SOURCE_DIR}"
        WORKING_DIRECTORY "${RENAMED_SOURCE_DIR}/.."
        DEPENDS sqlite
    )
else()
    ExternalProject_Add(
        sqlite
        URL "${DOWNLOAD_DIRECTORY}/${SQLITE3_PACKAGE}"
        CMAKE_ARGS
            -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
            -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
            -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
            -DBUILD_SHELL=ON
            -DSQLITE_ENABLE_RTREE=ON
            -DSQLITE_OMIT_DECLTYPE=OFF
    )
endif()

#
# CURL (PROJ dependency)
#
if(DOWNLOAD_ONLY)
    ExternalProject_Add(
        libcurl
        URL "${LIBCURL_URL}"
        DOWNLOAD_NAME "${LIBCURL_PACKAGE}"
        DOWNLOAD_DIR "${DOWNLOAD_DIRECTORY}"
        DOWNLOAD_NO_EXTRACT ON
        DOWNLOAD_NO_PROGRESS ON
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
    )
else()
    ExternalProject_Add(
        libcurl
        URL "${DOWNLOAD_DIRECTORY}/${LIBCURL_PACKAGE}"
        CMAKE_ARGS
            -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
            -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
            -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
            -DCURL_STATIC_CRT=ON
    )
endif()

# #
# # GEOS (GDAL dependency)
# #
# if(DOWNLOAD_ONLY)
# else()
#     ExternalProject_Add(
#         geos
#         GIT_REPOSITORY https://github.com/libgeos/geos.git
#         GIT_TAG        3.10.2
#         DEPENDS        libzlib libpng sqlite libtiff libcurl
#         CMAKE_ARGS
#             -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
#             -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
#             -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
#             # -DCURL_LIBRARY=CURL::libcurl
#         # UPDATE_COMMAND git reset --hard                                     # REMOVE ALL PATCHED CHANGES
#         # PATCH_COMMAND  git apply "${CMAKE_SOURCE_DIR}/patches/proj.diff"    # Re-patch
#     )
# endif()

# #
# # PROJ (GDAL dependency)
# #
# if(DOWNLOAD_ONLY)
# else()
#     ExternalProject_Add(
#         proj
#         GIT_REPOSITORY https://github.com/OSGeo/PROJ.git
#         GIT_TAG        9.0
#         DEPENDS        libzlib libpng sqlite libtiff libcurl
#         CMAKE_ARGS
#             -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
#             -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
#             -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
#             -DCURL_LIBRARY=CURL::libcurl
#         UPDATE_COMMAND git reset --hard                                     # REMOVE ALL PATCHED CHANGES
#         PATCH_COMMAND  git apply "${CMAKE_SOURCE_DIR}/patches/proj.diff"    # Re-patch
#     )
# endif()

# #
# # GDAL (OSG and osgEarth dependency)
# #
# if(DOWNLOAD_ONLY)
# else()
#     ExternalProject_Add(
#         gdal
#         GIT_REPOSITORY https://github.com/OSGeo/gdal.git
#         GIT_TAG        master    # v3.4.2
#         DEPENDS        libzlib libpng proj geos
#         CMAKE_ARGS
#             -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
#             -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
#             -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
#     )
# endif()

# #
# # OpenSceneGraph (OSG)
# #
# if(DOWNLOAD_ONLY)
# else()
#     ExternalProject_Add(
#         osg
#         GIT_REPOSITORY https://github.com/openscenegraph/OpenSceneGraph.git
#         GIT_TAG        OpenSceneGraph-3.6.5
#         DEPENDS        libzlib libpng gdal
#         CMAKE_ARGS
#         -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
#         -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
#         -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
#         -DCURL_LIBRARY=CURL::libcurl
#         UPDATE_COMMAND git reset --hard                                     # REMOVE ALL PATCHED CHANGES
#         PATCH_COMMAND  git apply "${CMAKE_SOURCE_DIR}/patches/osg.diff"    # Re-patch
#     )
# endif()

# #
# # osgQT
# #
# if(DOWNLOAD_ONLY)
# else()
#     ExternalProject_Add(
#         osgqt
#         GIT_REPOSITORY https://github.com/openscenegraph/osgQt.git
#         GIT_TAG        8fa9e2aed141488fa0818219f29e7ee9c7e667b0     # Dated 04/05/2021
#         DEPENDS        osg
#         CMAKE_ARGS
#         -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
#         -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
#         -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
#         -DBUILD_OSG_EXAMPLES=OFF
#         UPDATE_COMMAND git reset --hard                                     # REMOVE ALL PATCHED CHANGES
#         PATCH_COMMAND  git apply "${CMAKE_SOURCE_DIR}/patches/osgqt.diff"    # Re-patch
#     )
# endif()

# #
# # osgEarth
# #
# if(DOWNLOAD_ONLY)
# else()
#     ExternalProject_Add(
#         osgearth
#         GIT_REPOSITORY https://github.com/gwaldron/osgearth.git
#         GIT_TAG        3.1
#         DEPENDS        osg gdal osgqt
#         CMAKE_ARGS
#         -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
#         -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
#         -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
#         -DOSGEARTH_BUILD_EXAMPLES=OFF
#         -DOSGEARTH_BUILD_TESTS=OFF
#         -DOSGEARTH_BUILD_DOCS=OFF
#         -DOSGEARTH_BUILD_PROCEDURAL_NODEKIT=OFF
#         -DOSGEARTH_BUILD_TRITON_NODEKIT=OFF
#         -DOSGEARTH_BUILD_SILVERLINING_NODEKIT=OFF
#         -DOSG_DIR=${CMAKE_INSTALL_PREFIX}
#         UPDATE_COMMAND git reset --hard                                         # REMOVE ALL PATCHED CHANGES
#         PATCH_COMMAND  git apply "${CMAKE_SOURCE_DIR}/patches/osgearth.diff"    # Re-patch
#     )
# endif()

#
# A dummy project that forces the above to be built and installed
#
if (NOT DOWNLOAD_ONLY)
    add_library(dummy STATIC dummy.c)
    target_link_libraries(dummy)
    install(TARGETS dummy)
endif()
