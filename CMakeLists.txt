cmake_minimum_required(VERSION 3.9)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

project(File_Extractor VERSION 1.0.0 DESCRIPTION "File Extractor Library")

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)
include(CheckIncludeFile)

find_package(ZLIB REQUIRED)


set(FILE_EXTRACTOR_DEFINES)
set(FILE_EXTRACTOR_SOURCES
    fex/Binary_Extractor.cpp
    fex/blargg_common.cpp
    fex/blargg_errors.cpp
    fex/Data_Reader.cpp
    fex/fex.cpp
    fex/File_Extractor.cpp
    fex/Gzip_Extractor.cpp
    fex/Gzip_Reader.cpp
    fex/Rar_Extractor.cpp
    fex/Zip7_Extractor.cpp
    fex/Zip_Extractor.cpp
    fex/Zlib_Inflater.cpp
    7z_C/7zAlloc.c
    7z_C/7zBuf2.c
    7z_C/7zBuf.c
    7z_C/7zCrc.c
    7z_C/7zCrcOpt.c
    7z_C/7zDec.c
    7z_C/7zDecode.c
    7z_C/7zExtract.c
    7z_C/7zFile.c
    7z_C/7zIn.c
    7z_C/7zStream.c
    7z_C/Alloc.c
    7z_C/Bcj2.c
    7z_C/Bra86.c
    7z_C/Bra.c
    7z_C/BraIA64.c
    7z_C/CpuArch.c
    7z_C/Delta.c
    7z_C/LzFind.c
    7z_C/LzFindMt.c
    7z_C/Lzma2Dec.c
    7z_C/Lzma2Enc.c
    7z_C/Lzma86Dec.c
    7z_C/Lzma86Enc.c
    7z_C/LzmaDec.c
    7z_C/LzmaEnc.c
    7z_C/LzmaLib.c
    7z_C/MtCoder.c
    7z_C/Ppmd7.c
    7z_C/Ppmd7Dec.c
    7z_C/Ppmd7Enc.c
    unrar/archive.cpp
    unrar/arcread.cpp
    unrar/blake2s.cpp
    unrar/blake2sp.cpp
    unrar/blake2s_sse.cpp
    unrar/coder.cpp
    unrar/crc.cpp
    unrar/encname.cpp
    unrar/extract.cpp
    unrar/getbits.cpp
    unrar/hash.cpp
    unrar/headers.cpp
    unrar/model.cpp
    unrar/pathfn.cpp
    unrar/rarvm.cpp
    unrar/rarvmtbl.cpp
    unrar/rawread.cpp
    unrar/secpassword.cpp
    unrar/strfn.cpp
    unrar/suballoc.cpp
    unrar/timefn.cpp
    unrar/unicode.cpp
    unrar/unpack15.cpp
    unrar/unpack20.cpp
    unrar/unpack30.cpp
    unrar/unpack50.cpp
    unrar/unpack50frag.cpp
    unrar/unpack.cpp
    unrar/unpackinline.cpp
    unrar/unrar.cpp
    unrar/unrar_misc.cpp
    unrar/unrar_open.cpp
)
set(FILE_EXTRACTOR_HEADERS
    fex/fex.h
    fex/Binary_Extractor.h
    fex/Gzip_Extractor.h
    fex/blargg_source.h
    fex/blargg_endian.h
    fex/File_Extractor.h
    fex/Zip7_Extractor.h
    fex/Zlib_Inflater.h
    fex/blargg_errors.h
    fex/Data_Reader.h
    fex/Zip_Extractor.h
    fex/Rar_Extractor.h
    fex/Gzip_Reader.h
    fex/blargg_config.h
    fex/blargg_common.h
)

if(WIN32)
  set(FILE_EXTRACTOR_SOURCES ${FILE_EXTRACTOR_SOURCES}
    7z_C/win32/Threads.c
  )
else()
  set(FILE_EXTRACTOR_SOURCES ${FILE_EXTRACTOR_SOURCES}
    7z_C/posix/Threads.c
  )
  set(FILE_EXTRACTOR_DEFINES "_UNIX=1")
endif(WIN32)

check_include_file(stdint.h HAVE_STDINT_H)
if(HAVE_STDINT_H)
    list(APPEND FILE_EXTRACTOR_DEFINES HAVE_STDINT_H=1)
endif()

add_library(fex-object OBJECT ${FILE_EXTRACTOR_SOURCES})
set_target_properties(fex-object PROPERTIES POSITION_INDEPENDENT_CODE 1)
target_compile_definitions(fex-object PRIVATE ${FILE_EXTRACTOR_DEFINES})
target_include_directories(fex-object PRIVATE ${PROJECT_SOURCE_DIR})

add_library(fex SHARED)
target_link_libraries(fex PRIVATE fex-object)

add_library(fex-static STATIC)
target_link_libraries(fex-static PRIVATE fex-object)

set_target_properties(fex PROPERTIES VERSION ${PROJECT_VERSION})

target_include_directories(fex-object PUBLIC
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/fex>
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
)

target_include_directories(fex-static PUBLIC
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/fex>
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
)

target_include_directories(fex PUBLIC
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/fex>
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
)

target_link_libraries(fex PUBLIC ZLIB::ZLIB)
target_link_libraries(fex-static PUBLIC ZLIB::ZLIB)

install(TARGETS fex fex-static fex-object
  EXPORT fex-targets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(EXPORT fex-targets
  FILE
    "File_ExtractorTargets.cmake"
  NAMESPACE
    "File_Extractor::"
  DESTINATION
    "${CMAKE_INSTALL_LIBDIR}/cmake/File_Extractor"
)

write_basic_package_version_file(
    "${PROJECT_BINARY_DIR}/File_ExtractorConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(
    "${PROJECT_SOURCE_DIR}/cmake/File_ExtractorConfig.cmake.in"
    "${PROJECT_BINARY_DIR}/File_ExtractorConfig.cmake"
    INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/File_Extractor"
)

install(FILES
    "${PROJECT_BINARY_DIR}/File_ExtractorConfig.cmake"
    "${PROJECT_BINARY_DIR}/File_ExtractorConfigVersion.cmake"
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/File_Extractor"
)

configure_file(
    "${PROJECT_SOURCE_DIR}/cmake/libfex.pc.in"
    "${PROJECT_BINARY_DIR}/libfex.pc"
    @ONLY
)

install(FILES
    "${PROJECT_BINARY_DIR}/libfex.pc"
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig"
)

export(TARGETS fex fex-static fex-object
    FILE "${PROJECT_BINARY_DIR}/File_ExtractorTargets.cmake"
    NAMESPACE "File_Extractor::"
)
export(PACKAGE File_Extractor)

install(FILES ${FILE_EXTRACTOR_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/fex)

