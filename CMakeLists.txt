cmake_minimum_required(VERSION 3.10)

project(zlib
    LANGUAGES C
    VERSION 1.3.1.1)

option(ZLIB_BUILD_TESTING "Enable Zlib Examples as tests" ON)
option(ZLIB_BUILD_SHARED "Enable building zlib shared library" ON)
option(ZLIB_BUILD_STATIC "Enable building zlib static library" ON)
option(ZLIB_INSTALL_LIBRARIES "Enable installation of zlib" ON)
option(ZLIB_PREFIX
    "prefix for all types and library functions, see zconf.h.in"
    OFF)
mark_as_advanced(ZLIB_PREFIX)

if(WIN32)
    option(ZLIB_INSTALL_COMPAT_DLL "Install a copy as zlib1.dll" ON)
endif(WIN32)

get_property(IS_MULTI GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)

if(NOT DEFINED CMAKE_BUILD_TYPE AND NOT IS_MULTI)
    message(STATUS "No CMAKE_BUILD_TYPE set -- using Release")
    set(CMAKE_BUILD_TYPE Release)
endif(NOT DEFINED CMAKE_BUILD_TYPE AND NOT IS_MULTI)

include(CheckTypeSize)
include(CheckFunctionExists)
include(CheckIncludeFile)
include(CheckCSourceCompiles)
include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

#
# Check to see if we have large file support
#
set(CMAKE_REQUIRED_DEFINITIONS -D_LARGEFILE64_SOURCE=1)
check_type_size(off64_t OFF64_T)
set(CMAKE_REQUIRED_DEFINITIONS) # clear variable

#
# Check for fseeko
#
check_function_exists(fseeko HAVE_FSEEKO)

#
# Check for unistd.h
#
check_include_file(unistd.h Z_HAVE_UNISTD_H)

set(ZLIB_PC ${zlib_BINARY_DIR}/zlib.pc)
configure_file(${zlib_SOURCE_DIR}/zlib.pc.cmakein
               ${ZLIB_PC} @ONLY)
configure_file(${zlib_SOURCE_DIR}/zconf.h.cmakein
               ${zlib_BINARY_DIR}/zconf.h @ONLY)
include_directories(${zlib_BINARY_DIR} ${zlib_SOURCE_DIR})

#============================================================================
# zlib
#============================================================================

set(ZLIB_PUBLIC_HDRS
    ${zlib_BINARY_DIR}/zconf.h
    zlib.h)

set(ZLIB_PRIVATE_HDRS
    crc32.h
    deflate.h
    gzguts.h
    inffast.h
    inffixed.h
    inflate.h
    inftrees.h
    trees.h
    zutil.h)

set(ZLIB_SRCS
    adler32.c
    compress.c
    crc32.c
    deflate.c
    gzclose.c
    gzlib.c
    gzread.c
    gzwrite.c
    inflate.c
    infback.c
    inftrees.c
    inffast.c
    trees.c
    uncompr.c
    zutil.c)

set_source_files_properties(win32/zlib1.rc PROPERTIES
    COMPILE_FLAGS $<$<STREQUAL:${CMAKE_RC_OUTPUT_EXTENSION},.obj>:-DGCC_WINDRES>)

if(WIN32)
    set(zlib_static_suffix "s")
    set(CMAKE_DEBUG_POSTFIX "d")
endif(WIN32)

if(ZLIB_BUILD_SHARED)
    add_library(zlib SHARED
        ${ZLIB_SRCS}
        ${ZLIB_DLL_SRCS}
        ${ZLIB_PUBLIC_HDRS}
        ${ZLIB_PRIVATE_HDRS}
        $<$<OR:$<BOOL:${WIN32}>,$<BOOL:${CYGWIN}>>:win32/zlib1.rc>)
    add_library(ZLIB::ZLIB ALIAS zlib)
    target_include_directories(zlib PUBLIC
        $<BUILD_INTERFACE:${zlib_BINARY_DIR}>
        $<BUILD_INTERFACE:${zlib_SOURCE_DIR}>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
    target_compile_definitions(zlib
        PRIVATE
            $<$<BOOL:${HAVE_OFF64_T}>:_LARGEFILE64_SOURCE=1>
            $<$<BOOL:NOT:${HAVE_FSEEKO}>:NO_FSEEKO>
            $<$<BOOL:${MSVC}>:_CRT_SECURE_NO_DEPRECATE>
            $<$<BOOL:${MSVC}>:_CRT_NONSTDC_NO_DEPRECATE>)
    set(INSTALL_VERSION ${zlib_VERSION_MAJOR})

    if(NOT CYGWIN)
        set_target_properties(zlib PROPERTIES
            SOVERSION ${zlib_VERSION_MAJOR}
            VERSION ${INSTALL_VERSION})
    endif(NOT CYGWIN)

    set_target_properties(zlib PROPERTIES
        DEFINE_SYMBOL ZLIB_DLL
        EXPORT_NAME ZLIB
        OUTPUT_NAME z)
    if(UNIX AND NOT APPLE AND NOT(CMAKE_SYSTEM_NAME STREQUAL AIX))
        # On unix-like platforms the library is almost always called libz
        set_target_properties(zlib PROPERTIES
            LINK_FLAGS "-Wl,--version-script,\"${zlib_SOURCE_DIR}/zlib.map\"")
    endif(UNIX AND NOT APPLE AND NOT(CMAKE_SYSTEM_NAME STREQUAL AIX))
endif(ZLIB_BUILD_SHARED)

if(ZLIB_BUILD_STATIC)
    add_library(zlibstatic STATIC
        ${ZLIB_SRCS}
        ${ZLIB_PUBLIC_HDRS}
        ${ZLIB_PRIVATE_HDRS})
    add_library(ZLIB::ZLIBSTATIC ALIAS zlibstatic)
    target_include_directories(zlibstatic PUBLIC
        $<BUILD_INTERFACE:${zlib_BINARY_DIR}>
        $<BUILD_INTERFACE:${zlib_SOURCE_DIR}>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
    target_compile_definitions(zlibstatic
        PRIVATE
            $<$<BOOL:NOT:${HAVE_FSEEKO}>:NO_FSEEKO>
            $<$<BOOL:${MSVC}>:_CRT_SECURE_NO_DEPRECATE>
            $<$<BOOL:${MSVC}>:_CRT_NONSTDC_NO_DEPRECATE>
        PUBLIC
            $<$<BOOL:${HAVE_OFF64_T}>:_LARGEFILE64_SOURCE=1>)
    set_target_properties(zlibstatic PROPERTIES
        EXPORT_NAME ZLIBSTATIC
        OUTPUT_NAME z${zlib_static_suffix})
endif(ZLIB_BUILD_STATIC)

if(ZLIB_INSTALL_LIBRARIES)
    if(ZLIB_BUILD_SHARED)
        install(TARGETS zlib
            COMPONENT Runtime
            EXPORT zlibExport
            RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
            ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
            LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}")
        if(ZLIB_INSTALL_COMPAT_DLL)
            install(FILES  $<TARGET_FILE:zlib>
                COMPONENT Runtime
                RENAME zlib1.dll
                DESTINATION "${CMAKE_INSTALL_BINDIR}")
        endif(ZLIB_INSTALL_COMPAT_DLL)

        if(MSVC)
            install(FILES $<TARGET_PDB_FILE:zlib>
                COMPONENT Runtime
                DESTINATION ${CMAKE_INSTALL_BINDIR}
                CONFIGURATIONS Debug OR RelWithDebInfo
                OPTIONAL
            )
        endif(MSVC)
    endif(ZLIB_BUILD_SHARED)

    if(ZLIB_BUILD_STATIC)
        install(TARGETS zlibstatic
            COMPONENT Development
            EXPORT zlibExport
            RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
            ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
            LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}")
        if(ZLIB_INSTALL_COMPAT_DLL AND MINGW)
            install(FILES  $<TARGET_FILE:zlibstatic>
                COMPONENT Development
                RENAME libz.dll.a
                DESTINATION "${CMAKE_INSTALL_LIBDIR}")
        endif(ZLIB_INSTALL_COMPAT_DLL AND MINGW)

    endif(ZLIB_BUILD_STATIC)

    configure_package_config_file(${zlib_SOURCE_DIR}/zlibConfig.cmake.in
        ${zlib_BINARY_DIR}/zlibConfig.cmake
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/zlib
        NO_CHECK_REQUIRED_COMPONENTS_MACRO)

    write_basic_package_version_file(
        "${zlib_BINARY_DIR}/zlibConfigVersion.cmake"
        VERSION "${zlib_VERSION}"
        COMPATIBILITY AnyNewerVersion)

    install(FILES
        ${zlib_BINARY_DIR}/zlibConfig.cmake
        ${zlib_BINARY_DIR}/zlibConfigVersion.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/zlib)

    install(EXPORT zlibExport
        FILE zlib.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/zlib
        NAMESPACE ZLIB::)

    install(FILES ${ZLIB_PUBLIC_HDRS}
            COMPONENT Development
            DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")
    install(FILES zlib.3
            COMPONENT Development
            DESTINATION "${CMAKE_INSTALL_MANDIR}/man3")
    install(FILES ${ZLIB_PC}
            COMPONENT Development
            DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")
endif(ZLIB_INSTALL_LIBRARIES)

#============================================================================
# Example binaries
#============================================================================
if(ZLIB_BUILD_TESTING)
    enable_testing()

    if(ZLIB_BUILD_SHARED)
        add_executable(zlib_example test/example.c)
        target_link_libraries(zlib_example ZLIB::ZLIB)
        add_test(zlib_example zlib_example)

        add_executable(minigzip test/minigzip.c)
        target_link_libraries(minigzip ZLIB::ZLIB)

        if(HAVE_OFF64_T)
            add_executable(zlib_example64 test/example.c)
            target_compile_definitions(zlib_example64 PRIVATE LARGEFILE64_SOURCE=1)
            target_link_libraries(zlib_example64 ZLIB::ZLIB)
            add_test(zlib_example64 zlib_example64)

            add_executable(minigzip64 test/minigzip.c)
            target_compile_definitions(minigzip64 PRIVATE LARGEFILE64_SOURCE=1)
            target_link_libraries(minigzip64 ZLIB::ZLIB)
        endif(HAVE_OFF64_T)
    endif(ZLIB_BUILD_SHARED)

    if(ZLIB_BUILD_STATIC)
        add_executable(zlib_static_example test/example.c)
        target_link_libraries(zlib_static_example ZLIB::ZLIBSTATIC)
        add_test(zlib_static_example zlib_static_example)

        add_executable(static_minigzip test/minigzip.c)
        target_link_libraries(static_minigzip ZLIB::ZLIBSTATIC)

        if(HAVE_OFF64_T)
            add_executable(zlib_static_example64 test/example.c)
            target_compile_definitions(zlib_static_example64 PRIVATE LARGEFILE64_SOURCE=1)
            target_link_libraries(zlib_static_example64 ZLIB::ZLIBSTATIC)
            add_test(zlib_static_example64 zlib_static_example64)

            add_executable(static_minigzip64 test/minigzip.c)
            target_compile_definitions(static_minigzip64 PRIVATE LARGEFILE64_SOURCE=1)
            target_link_libraries(static_minigzip64 ZLIB::ZLIBSTATIC)
        endif(HAVE_OFF64_T)
    endif(ZLIB_BUILD_STATIC)

    add_subdirectory(test)
endif(ZLIB_BUILD_TESTING)
