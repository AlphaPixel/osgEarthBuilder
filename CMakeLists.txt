cmake_minimum_required(VERSION 3.20)

find_program(PATCH_COMMAND 
    NAMES patch patch.exe 
    REQUIRED
    PATHS 
        "c:/program files/git/usr/bin"
)

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
set(LIBZLIB_HASH "SHA256=7c9917b80c125e317e44c9d83a4c15390dac9967df27397024beaf4342e0f8cd")

set(LIBTIFF_VERSION "4.3.0")
set(LIBTIFF_URL "https://gitlab.com/libtiff/libtiff/-/archive/v${LIBTIFF_VERSION}/libtiff-v${LIBTIFF_VERSION}.zip")
set(LIBTIFF_PACKAGE "libtiff-v${LIBTIFF_VERSION}.zip")
set(LIBTIFF_HASH "SHA256=4fca1b582c88319f3ad6ecd5b46320eadaf5eb4ef6f6c32d44caaae4a03d0726")

set(LIBPNG_VERSION "1.6.35")
set(LIBPNG_URL "https://github.com/glennrp/libpng/archive/refs/tags/v${LIBPNG_VERSION}.zip")
set(LIBPNG_PACKAGE "libpng-v${LIBPNG_VERSION}.zip")
set(LIBPNG_HASH "SHA256=3d22d46c566b1761a0e15ea397589b3a5f36ac09b7c785382e6470156c04247f")

set(SQLITE3_VERSION "3.38.2")
set(SQLITE3_URL "https://github.com/azadkuh/sqlite-amalgamation")  # this is a GIT repository since no prebuild source packages are available.
set(SQLITE3_PACKAGE "sqlite-v${SQLITE3_VERSION}.zip")
#set(SQLLITE3_HASH "")  No hash calculated here since SQLITE is pulled from GIT and a package created manually.

set(CURL_VERSION "7.82.0")
string(REPLACE "." "_" CURL_VERSION_UNDERSCORE ${CURL_VERSION})
set(CURL_URL "https://github.com/curl/curl/releases/download/curl-${CURL_VERSION_UNDERSCORE}/curl-${CURL_VERSION}.zip")
set(CURL_PACKAGE "curl-v${CURL_VERSION}.zip")
set(CURL_HASH "SHA256=58a06d7788abbf1c4d326b37f6826d9089b7167187144b6cf2ef2c1851196b90")

set(GEOS_VERSION "3.10.2")
set(GEOS_URL "https://github.com/libgeos/geos/archive/refs/tags/${GEOS_VERSION}.zip")
set(GEOS_PACKAGE "geos-v${GEOS_VERSION}.zip")
set(GEOS_HASH "SHA256=230800ee13172876a3511d3c393a64815eb0b3ff0c435bf2bbf71e1e31e6d715")

set(PROJ_VERSION "9.0.0")
set(PROJ_URL "https://github.com/OSGeo/PROJ/archive/refs/tags/${PROJ_VERSION}.zip")
set(PROJ_PACKAGE "proj-v${PROJ_VERSION}.zip")
set(PROJ_HASH "SHA256=ceb2934494543fad738f5aec10752f679c9de5599e76d2c4c49f1bb900b1ce1b")

set(GDAL_VERSION "de5c6b3dc4016c2abf055a9c7281e17bdad53e73")   # Master branch as of 2022/4/12 - needed to build using CMake
set(GDAL_URL "https://github.com/OSGeo/gdal.git")
set(GDAL_PACKAGE "gdal-v${GDAL_VERSION}.zip")
set(GDAL_HASH "SHA256=7e12a88acf52a3fef55f34cd473d23aef37bb4261eefbd358d1e8d1aea5c05a5")

set(OSG_VERSION "3.6.5")
set(OSG_URL "https://github.com/openscenegraph/OpenSceneGraph/archive/refs/tags/OpenSceneGraph-${OSG_VERSION}.zip")
set(OSG_PACKAGE "osg-v${OSG_VERSION}.zip")
set(OSG_HASH "SHA256=0e9e3e4cc6f463f21a901934a95e9264b231a1d5db90f72dcb4b8cc94b0d1b3b")

set(OSGQT_VERSION "8fa9e2aed141488fa0818219f29e7ee9c7e667b0")   # Master branch as of 2021/4/5
set(OSGQT_URL "https://github.com/openscenegraph/osgQt.git")
set(OSGQT_PACKAGE "osgqt-v${OSGQT_VERSION}.zip")
set(OSGQT_HASH "SHA256=1c61bea6a9f6b61b5c29597a678790d15007d4266afc87193de7afcfba8428ca")

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
        URL_HASH "${LIBZLIB_HASH}"
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
        URL_HASH "${LIBZLIB_HASH}"
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
        URL_HASH "${LIBTIFF_HASH}"
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
        URL_HASH "${LIBTIFF_HASH}"
        DEPENDS libzlib
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
        URL_HASH "${LIBPNG_HASH}"
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
        URL_HASH "${LIBPNG_HASH}"
        DEPENDS libzlib
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
        GIT_TAG ${SQLITE3_VERSION}
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
        UPDATE_DISCONNECTED YES
    )

    ExternalProject_Get_Property(sqlite SOURCE_DIR)

    add_custom_target(sqlite_zip ALL
        COMMAND "${CMAKE_COMMAND}" -E rm "-rf" "${SOURCE_DIR}/.git*"
        COMMAND "${CMAKE_COMMAND}" -E tar "cfv" "${DOWNLOAD_DIRECTORY}/${SQLITE3_PACKAGE}" --format=zip "${SOURCE_DIR}"
        WORKING_DIRECTORY "${SOURCE_DIR}/.."
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
        curl
        URL "${CURL_URL}"
        URL_HASH "${CURL_HASH}"
        DOWNLOAD_NAME "${CURL_PACKAGE}"
        DOWNLOAD_DIR "${DOWNLOAD_DIRECTORY}"
        DOWNLOAD_NO_EXTRACT ON
        DOWNLOAD_NO_PROGRESS ON
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
    )
else()
    ExternalProject_Add(
        curl
        URL "${DOWNLOAD_DIRECTORY}/${CURL_PACKAGE}"
        URL_HASH "${CURL_HASH}"
        DEPENDS libzlib
        CMAKE_ARGS
            -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
            -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
            -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
            -DCURL_STATIC_CRT=ON
    )
endif()

#
# GEOS (GDAL dependency)
#
if(DOWNLOAD_ONLY)
    ExternalProject_Add(
        geos
        URL "${GEOS_URL}"
        URL_HASH "${GEOS_HASH}"
        DOWNLOAD_NAME "${GEOS_PACKAGE}"
        DOWNLOAD_DIR "${DOWNLOAD_DIRECTORY}"
        DOWNLOAD_NO_EXTRACT ON
        DOWNLOAD_NO_PROGRESS ON
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
    )
    else()
    ExternalProject_Add(
        geos
        URL "${DOWNLOAD_DIRECTORY}/${GEOS_PACKAGE}"
        URL_HASH "${GEOS_HASH}"
        DEPENDS libzlib libpng sqlite libtiff curl
        CMAKE_ARGS
            -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
            -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
            -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    )
endif()

#
# PROJ (GDAL dependency)
#
if(DOWNLOAD_ONLY)
    ExternalProject_Add(
        proj
        URL "${PROJ_URL}"
        URL_HASH "${PROJ_HASH}"
        DOWNLOAD_NAME "${PROJ_PACKAGE}"
        DOWNLOAD_DIR "${DOWNLOAD_DIRECTORY}"
        DOWNLOAD_NO_EXTRACT ON
        DOWNLOAD_NO_PROGRESS ON
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
    )
else()
    ExternalProject_Add(
        proj
        URL "${DOWNLOAD_DIRECTORY}/${PROJ_PACKAGE}"
        URL_HASH "${PROJ_HASH}"
        DEPENDS libzlib libpng sqlite libtiff curl
        CMAKE_ARGS
            -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
            -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
            -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
            -DCURL_LIBRARY=CURL::libcurl
            -DBUILD_TESTING=OFF
        PATCH_COMMAND "${PATCH_COMMAND}" -p1 -i "${CMAKE_SOURCE_DIR}/patches/proj.diff"
    )
endif()

#
# GDAL (OSG and osgEarth dependency)
#
if(DOWNLOAD_ONLY)
    ExternalProject_Add(
        gdal
        GIT_REPOSITORY "${GDAL_URL}"
        GIT_TAG ${GDAL_VERSION}
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
        UPDATE_DISCONNECTED YES
    )

    ExternalProject_Get_Property(gdal SOURCE_DIR)

    add_custom_target(gdal_zip ALL
        COMMAND "${CMAKE_COMMAND}" -E rm "-rf" "${SOURCE_DIR}/.git*"
        COMMAND "${CMAKE_COMMAND}" -E tar "cfv" "${DOWNLOAD_DIRECTORY}/${GDAL_PACKAGE}" --format=zip "${SOURCE_DIR}"
        WORKING_DIRECTORY "${SOURCE_DIR}/.."
        DEPENDS gdal
    )
else()
    # NOTE: The GDAL patch is from 3.4.2 (commit 649825c2979beab0fc16cc03b74e14b1780acb28) -> master (commit de5c6b3dc4016c2abf055a9c7281e17bdad53e73 - dated 2022/4/12)
    #       This patch is necessary to be able to build GDAL with CMake.
    ExternalProject_Add(
        gdal
        URL "${DOWNLOAD_DIRECTORY}/${GDAL_PACKAGE}"
        URL_HASH "${GDAL_HASH}"
        DEPENDS libzlib libpng proj geos
        CMAKE_ARGS
            -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
            -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
            -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    )
endif()

#
# OpenSceneGraph (OSG)
#
if(DOWNLOAD_ONLY)
    ExternalProject_Add(
        osg
        URL "${OSG_URL}"
        URL_HASH "${OSG_HASH}"
        DOWNLOAD_NAME "${OSG_PACKAGE}"
        DOWNLOAD_DIR "${DOWNLOAD_DIRECTORY}"
        DOWNLOAD_NO_EXTRACT ON
        DOWNLOAD_NO_PROGRESS ON
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
    )
else()
    ExternalProject_Add(
        osg
        URL "${DOWNLOAD_DIRECTORY}/${OSG_PACKAGE}"
        URL_HASH "${OSG_HASH}"
        DEPENDS libzlib libpng gdal
        CMAKE_ARGS
            -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
            -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
            -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
            -DCURL_LIBRARY=CURL::libcurl
        PATCH_COMMAND "${PATCH_COMMAND}" -p1 -i "${CMAKE_SOURCE_DIR}/patches/osg.diff"
    )
endif()

#
# osgQT
#
if(DOWNLOAD_ONLY)
    ExternalProject_Add(
        osgqt
        GIT_REPOSITORY "${OSGQT_URL}"
        GIT_TAG ${OSGQT_VERSION}
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
        UPDATE_DISCONNECTED YES
    )

    ExternalProject_Get_Property(osgqt SOURCE_DIR)

    add_custom_target(osgqt_zip ALL
        COMMAND "${CMAKE_COMMAND}" -E rm "-rf" "${SOURCE_DIR}/.git*"
        COMMAND "${CMAKE_COMMAND}" -E tar "cfv" "${DOWNLOAD_DIRECTORY}/${OSGQT_PACKAGE}" --format=zip "${SOURCE_DIR}"
        WORKING_DIRECTORY "${SOURCE_DIR}/.."
        DEPENDS osgt
    )
else()
    ExternalProject_Add(
        osgqt
        URL "${DOWNLOAD_DIRECTORY}/${OSGQT_PACKAGE}"
        URL_HASH "${OSGQT_HASH}"
        DEPENDS osg
        CMAKE_ARGS
            -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
            -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
            -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
            -DBUILD_OSG_EXAMPLES=OFF
        PATCH_COMMAND "${PATCH_COMMAND}" -p1 -i "${CMAKE_SOURCE_DIR}/patches/osgqt.diff"
    )
endif()

# #
# # osgEarth
# #
# if(DOWNLOAD_ONLY)
# else()
#     ExternalProject_Add(
#         osgearth
#         GIT_REPOSITORY https://github.com/gwaldron/osgearth.git
#         GIT_TAG        3.1
#         DEPENDS osg gdal osgqt
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
