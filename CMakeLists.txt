cmake_minimum_required(VERSION 3.20)

project(osgEarthBuilder)

if ("${CMAKE_BUILD_TYPE}" STREQUAL "")
    set(CMAKE_BUILD_TYPE "Debug")
endif()

# Add the install prefix to CMAKE_PREFIX_PATH
list(APPEND CMAKE_PREFIX_PATH "${CMAKE_INSTALL_PREFIX}"})

# include(FetchContent)
include(ExternalProject)

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
ExternalProject_Add(
    libzlib
    GIT_REPOSITORY https://github.com/madler/zlib.git
    GIT_TAG        v1.2.12
    CMAKE_ARGS
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
)

#
# libTIFF (PROJ/OSG dependency)
#
ExternalProject_Add(
    libtiff
    GIT_REPOSITORY https://gitlab.com/libtiff/libtiff.git
    GIT_TAG        v4.3.0
    DEPENDS        libzlib
    CMAKE_ARGS
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
        -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
)

#
# libPNG (OSG dependency)
#
ExternalProject_Add(
    libpng
    GIT_REPOSITORY https://github.com/glennrp/libpng.git
    GIT_TAG        v1.6.37
    DEPENDS        libzlib
    CMAKE_ARGS
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
        -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
)

#
# SQLite (PROJ dependency)
#
ExternalProject_Add(
    sqlite
    GIT_REPOSITORY https://github.com/azadkuh/sqlite-amalgamation
    GIT_TAG        3.38.2
    CMAKE_ARGS
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
        -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
        -DBUILD_SHELL=YES
)

#
# CURL (PROJ dependency)
#
ExternalProject_Add(
    libcurl
    GIT_REPOSITORY https://github.com/curl/curl.git
    GIT_TAG        curl-7_82_0
    CMAKE_ARGS
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
        -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
        -DCURL_STATIC_CRT=ON
)

#
# PROJ (GDAL dependency)
#
ExternalProject_Add(
    proj
    GIT_REPOSITORY https://github.com/OSGeo/PROJ.git
    GIT_TAG        9.0
    DEPENDS        libzlib libpng sqlite libtiff libcurl
    CMAKE_ARGS
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
        -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
        -DCURL_LIBRARY=CURL::libcurl
    UPDATE_COMMAND git reset --hard                                     # REMOVE ALL PATCHED CHANGES
    PATCH_COMMAND  git apply "${CMAKE_SOURCE_DIR}/patches/proj.diff"    # Re-patch
)

#
# GDAL (OSG and osgEarth dependency)
#
ExternalProject_Add(
    gdal
    GIT_REPOSITORY https://github.com/OSGeo/gdal.git
    GIT_TAG        master    # v3.4.2
    DEPENDS        libzlib libpng proj
    CMAKE_ARGS
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
        -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
)

#
# OpenSceneGraph (OSG)
#
ExternalProject_Add(
    osg
    GIT_REPOSITORY https://github.com/openscenegraph/OpenSceneGraph.git
    GIT_TAG        OpenSceneGraph-3.6.5
    DEPENDS        libzlib libpng gdal
    CMAKE_ARGS
       -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
       -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
       -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
       -DCURL_LIBRARY=CURL::libcurl
)

#
# osgQT
#
ExternalProject_Add(
    osgqt
    GIT_REPOSITORY https://github.com/openscenegraph/osgQt.git
    GIT_TAG        8fa9e2aed141488fa0818219f29e7ee9c7e667b0     # Dated 04/05/2021
    DEPENDS        osg
    CMAKE_ARGS
       -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
       -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
       -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
       -DBUILD_OSG_EXAMPLES=OFF
    UPDATE_COMMAND git reset --hard                                     # REMOVE ALL PATCHED CHANGES
    PATCH_COMMAND  git apply "${CMAKE_SOURCE_DIR}/patches/osgqt.diff"    # Re-patch
)

#
# A dummy project that forces the above to be built and installed
#
#add_custom_target(dummy)

add_library(dummy STATIC dummy.c)
target_link_libraries(dummy)
install(TARGETS dummy)
