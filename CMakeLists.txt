cmake_minimum_required(VERSION 3.5)

project(zlib
    LANGUAGES C
    VERSION 1.3.1.1)

option(ZLIB_BUILD_TESTING "Enable Zlib Examples as tests" ON)
option(ZLIB_BUILD_SHARED "Enable building zlib shared library" ON)
option(ZLIB_BUILD_STATIC "Enable building zlib static library" ON)
option(ZLIB_INSTALL_LIBRARIES "Enable installation of zlib" ON)

if(WIN32)
    option(ZLIB_INSTALL_COMPAT_DLL "Install a copy as zlib1.dll" ON)
endif(WIN32)

include(CheckTypeSize)
include(CheckFunctionExists)
include(CheckIncludeFile)
include(CheckCSourceCompiles)
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
    target_include_directories(zlib PUBLIC
        $<BUILD_INTERFACE:${zlib_BINARY_DIR}>
        $<BUILD_INTERFACE:${zlib_SOURCE_DIR}>)
    target_compile_definitions(zlib
        PRIVATE
            $<$<BOOL:${HAVE_OFF64_T}>:_LARGEFILE64_SOURCE=1>
            $<$<BOOL:NOT:${HAVE_FSEEKO}>:NO_FSEEKO>
            $<$<BOOL:${MSVC}>:_CRT_SECURE_NO_DEPRECATE>
            $<$<BOOL:${MSVC}>:_CRT_NONSTDC_NO_DEPRECATE>)
    # The VERSION property causes shared libraries on Linux to have the full version
    # encoded into their final filename.  We disable this on Cygwin because
    # it causes cygz-${ZLIB_FULL_VERSION}.dll to be created when cygz.dll
    # seems to be the default.
    #
    # This has no effect with MSVC, on that platform the version info for
    # the DLL comes from the resource file win32/zlib1.rc
    set_target_properties(zlib PROPERTIES
        DEFINE_SYMBOL ZLIB_DLL
        OUTPUT_NAME z
        SOVERSION 1
        $<$<BOOL:NOT:$CYGWIN}>:VERSION ${zlib_VERSION}>)
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
    target_include_directories(zlibstatic PUBLIC
        $<BUILD_INTERFACE:${zlib_BINARY_DIR}>
        $<BUILD_INTERFACE:${zlib_SOURCE_DIR}>)
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
            RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
            ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
            LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}")
    endif(ZLIB_BUILD_STATIC)

    install(FILES ${ZLIB_PUBLIC_HDRS}
            COMPONENT Development
            DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
        FILES zlib.3
            COMPONENT Development
            DESTINATION "${CMAKE_INSTALL_MANDIR}/man3"
        FILES ${ZLIB_PC}
            COMPONENT Development
            DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")
endif(ZLIB_INSTALL_LIBRARIES)

#============================================================================
# Example binaries
#============================================================================
if(ZLIB_BUILD_TESTING)
    enable_testing()
    add_executable(zlib_example test/example.c)
    target_link_libraries(zlib_example zlib)
    add_test(zlib_example zlib_example)

    add_executable(minigzip test/minigzip.c)
    target_link_libraries(minigzip zlib)

    if(HAVE_OFF64_T)
        add_executable(zlib_example64 test/example.c)
        target_compile_definitions(zlib_example64 PRIVATE LARGEFILE64_SOURCE=1)
        target_link_libraries(zlib_example64 zlib)
        add_test(zlib_example64 zlib_example64)

        add_executable(minigzip64 test/minigzip.c)
        target_compile_definitions(minigzip64 PRIVATE LARGEFILE64_SOURCE=1)
        target_link_libraries(minigzip64 zlib)
    endif(HAVE_OFF64_T)
endif(ZLIB_BUILD_TESTING)
