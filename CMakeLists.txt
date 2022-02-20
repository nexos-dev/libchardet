# CMakeLists.txt - contains build system

cmake_minimum_required(VERSION 3.00)
project(libchardet VERSION 1.0.7)
enable_language(C CXX)

option(BUILD_SHARED_LIBS "Specifies if shared libraries should be built" OFF)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 11)

include(GNUInstallDirs)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

include(SdkCompilerTest)
include(CMakePackageConfigHelpers)

# Check for stat
include(CheckSymbolExists)
check_symbol_exists(stat "sys/stat.h" HAVE_UNIX_STAT)
if(NOT HAVE_UNIX_STAT)
    check_symbol_exists(_stat "sys/stat.h" HAVE_WIN32_STAT)
endif()

# Configure everything
configure_file(${CMAKE_SOURCE_DIR}/chardet-config.h.in ${CMAKE_BINARY_DIR}/chardet-config.h)
configure_file(${CMAKE_SOURCE_DIR}/chardet.pc.in ${CMAKE_BINARY_DIR}/chardet.pc @ONLY)

# Sources
list(APPEND CHARDET_SOURCES
            src/chardet.cpp
            src/CharDistribution.cpp
            src/JpCntx.cpp
            src/nsBig5Prober.cpp
            src/nsCharSetProber.cpp
            src/nsEscCharsetProber.cpp
            src/nsEscSM.cpp
            src/nsEUCJPProber.cpp
            src/nsEUCKRProber.cpp
            src/nsEUCTWProber.cpp
            src/nsGB2312Prober.cpp
            src/nsHebrewProber.cpp
            src/nsLatin1Prober.cpp
            src/nsMBCSGroupProber.cpp
            src/nsMBCSSM.cpp
            src/nsSBCharSetProber.cpp
            src/nsSBCSGroupProber.cpp
            src/nsSJISProber.cpp
            src/nsUniversalDetector.cpp
            src/nsUTF8Prober.cpp
            src/tables/LangArabicModel.cpp
            src/tables/LangBulgarianModel.cpp
            src/tables/LangCyrillicModel.cpp
            src/tables/LangDanishModel.cpp
            src/tables/LangEsperantoModel.cpp
            src/tables/LangFrenchModel.cpp
            src/tables/LangGermanModel.cpp
            src/tables/LangGreekModel.cpp
            src/tables/LangHebrewModel.cpp
            src/tables/LangHungarianModel.cpp
            src/tables/LangSpanishModel.cpp
            src/tables/LangThaiModel.cpp
            src/tables/LangTurkishModel.cpp
            src/tables/LangVietnameseModel.cpp)

# Headers
list(APPEND CHARDET_HEADERS 
            src/nsUniversalDetector.h
            src/chardet.h
            include/nscore.h
            include/version.h
            ${CMAKE_BINARY_DIR}/chardet-config.h)

list(APPEND CHARDET_HEADER_DIRS
            ${CMAKE_SOURCE_DIR}/include
            ${CMAKE_SOURCE_DIR}/src
            ${CMAKE_BINARY_DIR})

# Create the library
add_library(chardet ${CHARDET_SOURCES})

# Setup compiler flags
target_compile_definitions(chardet PRIVATE IN_CHARDET)
target_compile_options(chardet PRIVATE -nostdinc++ -fno-rtti -fno-exceptions)
target_include_directories(chardet PUBLIC ${CHARDET_HEADER_DIRS})

# Set SO name
set_target_properties(chardet PROPERTIES VERSION ${PROJECT_VERSION} SOVERSION ${PROJECT_VERSION})

# Ensure symbols are hidden by default
if(BUILD_SHARED_LIBS)
    set_target_properties(nex PROPERTIES C_VISIBILITY_PRESET hidden)
endif()

# Check for a visibility method
check_library_visibility(HAVE_DECLSPEC_EXPORT HAVE_VISIBILITY)

# Install everything
install(FILES ${CMAKE_BINARY_DIR}/chardet.pc DESTINATION ${CMAKE_INSTALL_DATADIR}/pkgconfig)
install(FILES ${CHARDET_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/chardet)
install(TARGETS chardet)

# Install man pages
if(UNIX)
    install(DIRECTORY ${CMAKE_SOURCE_DIR}/man/ DESTINATION ${CMAKE_INSTALL_MANDIR})
endif()

# Figure out library type
if(BUILD_SHARED_LIBS)
    set(LIBCHARDET_LIBTYPE "SHARED")
    set(LIBCHARDET_LIBRARY_FILE 
        "${CMAKE_INSTALL_FULL_LIBDIR}/${CMAKE_SHARED_LIBRARY_PREFIX}chardet.${CMAKE_SHARED_LIBRARY_SUFFIX}")
else()
    set(LIBCHARDET_LIBTYPE "STATIC")
    set(LIBCHARDET_LIBRARY_FILE 
        "${CMAKE_INSTALL_FULL_LIBDIR}/${CMAKE_STATIC_LIBRARY_PREFIX}chardet.${CMAKE_STATIC_LIBRARY_SUFFIX}")
endif()

# Configure package configuration file
configure_package_config_file(${CMAKE_SOURCE_DIR}/LibChardetConfig.cmake.in 
                              ${CMAKE_BINARY_DIR}/LibChardetConfig.cmake
                              INSTALL_DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/cmake
                              PATH_VARS CMAKE_INSTALL_INCLUDEDIR)

write_basic_package_version_file(${CMAKE_BINARY_DIR}/LibChardetConfigVersion.cmake
                                 COMPATIBILITY AnyNewerVersion)

# Install cmake package files
install(FILES ${CMAKE_BINARY_DIR}/LibChardetConfig.cmake 
        DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/cmake)
install(FILES ${CMAKE_BINARY_DIR}/LibChardetConfigVersion.cmake 
        DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/cmake)
