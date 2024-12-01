cmake_minimum_required(VERSION 3.5)
set(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS ON)

project(zlib
    LANGUAGES C
    VERSION 1.3.1.1)

option(ZLIB_BUILD_EXAMPLES "Enable Zlib Examples" ON)
option(ZLIB_BUILD_SHARED "Enable building zlib shared library" ON)
option(ZLIB_BUILD_STATIC "Enable building zlib static library" ON)

if(WIN32)
    option(ZLIB_INSTALL_COMPAT_DLL "Install a copy as zlib1.dll" ON)
endif(WIN32)

include(CheckTypeSize)
include(CheckFunctionExists)
include(CheckIncludeFile)
include(CheckCSourceCompiles)
include(GNUInstallDirs)
enable_testing()

#
# Check to see if we have large file support
#
set(CMAKE_REQUIRED_DEFINITIONS -D_LARGEFILE64_SOURCE=1)

check_type_size(off64_t OFF64_T)
if(HAVE_OFF64_T)
   add_definitions(-D_LARGEFILE64_SOURCE=1)
endif()
set(CMAKE_REQUIRED_DEFINITIONS) # clear variable

#
# Check for fseeko
#
check_function_exists(fseeko HAVE_FSEEKO)
if(NOT HAVE_FSEEKO)
    add_definitions(-DNO_FSEEKO)
endif()

#
# Check for unistd.h
#
check_include_file(unistd.h Z_HAVE_UNISTD_H)

if(MSVC)
    add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
    add_definitions(-D_CRT_NONSTDC_NO_DEPRECATE)
    include_directories(${zlib_SOURCE_DIR})
endif()

set(ZLIB_PC ${zlib_BINARY_DIR}/zlib.pc)
configure_file( ${zlib_SOURCE_DIR}/zlib.pc.cmakein
		${ZLIB_PC} @ONLY)
configure_file(	${zlib_SOURCE_DIR}/zconf.h.cmakein
		${zlib_BINARY_DIR}/zconf.h @ONLY)
include_directories(${zlib_BINARY_DIR} ${zlib_SOURCE_DIR})


#============================================================================
# zlib
#============================================================================

set(ZLIB_PUBLIC_HDRS
    ${zlib_BINARY_DIR}/zconf.h
    zlib.h
)
set(ZLIB_PRIVATE_HDRS
    crc32.h
    deflate.h
    gzguts.h
    inffast.h
    inffixed.h
    inflate.h
    inftrees.h
    trees.h
    zutil.h
)
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
    zutil.c
)

if(NOT MINGW)
    set(ZLIB_DLL_SRCS
        win32/zlib1.rc # If present will override custom build rule below.
    )
endif()

# parse the full version number from zlib.h and include in ZLIB_FULL_VERSION
file(READ ${zlib_SOURCE_DIR}/zlib.h _zlib_h_contents)
string(REGEX REPLACE ".*#define[ \t]+ZLIB_VERSION[ \t]+\"([-0-9A-Za-z.]+)\".*"
    "\\1" ZLIB_VERSION ${_zlib_h_contents})
if(MINGW)
    # This gets us DLL resource information when compiling on MinGW.
    if(NOT CMAKE_RC_COMPILER)
        set(CMAKE_RC_COMPILER windres.exe)
    endif()

    add_custom_command(OUTPUT ${zlib_BINARY_DIR}/zlib1rc.obj
                       COMMAND ${CMAKE_RC_COMPILER}
                            -D GCC_WINDRES
                            -I ${zlib_SOURCE_DIR}
                            -I ${zlib_BINARY_DIR}
                            -o ${zlib_BINARY_DIR}/zlib1rc.obj
                            -i ${zlib_SOURCE_DIR}/win32/zlib1.rc)
    set(ZLIB_DLL_SRCS ${zlib_BINARY_DIR}/zlib1rc.obj)
endif(MINGW)

if(WIN32)
    set (zlib_static_suffix "s")
    set (zlib_debug_suffix "d")
endif(WIN32)

if(ZLIB_BUILD_SHARED)
    add_library(zlib SHARED ${ZLIB_SRCS} ${ZLIB_DLL_SRCS} ${ZLIB_PUBLIC_HDRS} ${ZLIB_PRIVATE_HDRS})
    target_include_directories(zlib PUBLIC
        $<BUILD_INTERFACE:${zlib_BINARY_DIR}>
        $<BUILD_INTERFACE:${zlib_SOURCE_DIR}>)
    set_target_properties(zlib PROPERTIES
        DEFINE_SYMBOL ZLIB_DLL
        SOVERSION 1
        PROPERTIES OUTPUT_NAME z
        PROPERTIES OUTPUT_NAME_DEBUG z${zlib_debug_suffix})
    if(UNIX AND NOT APPLE AND NOT(CMAKE_SYSTEM_NAME STREQUAL AIX))
        # On unix-like platforms the library is almost always called libz
        set_target_properties(zlib PROPERTIES
            LINK_FLAGS "-Wl,--version-script,\"${zlib_SOURCE_DIR}/zlib.map\"")
    endif(UNIX AND NOT APPLE AND NOT(CMAKE_SYSTEM_NAME STREQUAL AIX))
endif(ZLIB_BUILD_SHARED)

if(ZLIB_BUILD_STATIC)
    add_library(zlibstatic STATIC ${ZLIB_SRCS} ${ZLIB_PUBLIC_HDRS} ${ZLIB_PRIVATE_HDRS})
    target_include_directories(zlibstatic PUBLIC
        $<BUILD_INTERFACE:${zlib_BINARY_DIR}>
        $<BUILD_INTERFACE:${zlib_SOURCE_DIR}>)
    set_target_properties(zlibstatic PROPERTIES
        PROPERTIES OUTPUT_NAME z${zlib_static_suffix}
        PROPERTIES OUTPUT_NAME_DEBUG z${zlib_static_suffix}${zlib_debug_suffix})
endif(ZLIB_BUILD_STATIC)

if(NOT CYGWIN AND ZLIB_BUILD_SHARED)
    # This property causes shared libraries on Linux to have the full version
    # encoded into their final filename.  We disable this on Cygwin because
    # it causes cygz-${ZLIB_FULL_VERSION}.dll to be created when cygz.dll
    # seems to be the default.
    #
    # This has no effect with MSVC, on that platform the version info for
    # the DLL comes from the resource file win32/zlib1.rc
    set_target_properties(zlib PROPERTIES VERSION ${zlib_VERSION})
endif()

if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL )
    if(ZLIB_BUILD_SHARED)
        install(TARGETS zlib
            RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
            ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
            LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}")
        if(ZLIB_INSTALL_COMPAT_DLL)
            install(FILES  $<TARGET_FILE:zlib>
                RENAME zlib1.dll
                DESTINATION "${CMAKE_INSTALL_BINDIR}")
        endif(ZLIB_INSTALL_COMPAT_DLL)

        if(MSVC)
            install(FILES $<TARGET_PDB_FILE:zlib>
                DESTINATION ${CMAKE_INSTALL_BINDIR}
                CONFIGURATIONS Debug OR RelWithDebInfo
                OPTIONAL
            )
        endif(MSVC)

    endif(ZLIB_BUILD_SHARED)

    if(ZLIB_BUILD_STATIC)
        install(TARGETS zlibstatic
            RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
            ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
            LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}")
    endif(ZLIB_BUILD_STATIC)
endif()
if(NOT SKIP_INSTALL_HEADERS AND NOT SKIP_INSTALL_ALL )
    install(FILES ${ZLIB_PUBLIC_HDRS} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")
endif()
if(NOT SKIP_INSTALL_FILES AND NOT SKIP_INSTALL_ALL )
    install(FILES zlib.3 DESTINATION "${CMAKE_INSTALL_MANDIR}/man3")
endif()
if(NOT SKIP_INSTALL_FILES AND NOT SKIP_INSTALL_ALL )
    install(FILES ${ZLIB_PC} DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")
endif()

#============================================================================
# Example binaries
#============================================================================
if(ZLIB_BUILD_EXAMPLES)
    add_executable(zlib_example test/example.c)
    target_link_libraries(zlib_example zlib)
    add_test(zlib_example zlib_example)

    add_executable(minigzip test/minigzip.c)
    target_link_libraries(minigzip zlib)

    if(HAVE_OFF64_T)
        add_executable(zlib_example64 test/example.c)
        target_link_libraries(zlib_example64 zlib)
        set_target_properties(zlib_example64 PROPERTIES COMPILE_FLAGS "-D_FILE_OFFSET_BITS=64")
        add_test(zlib_example64 zlib_example64)

        add_executable(minigzip64 test/minigzip.c)
        target_link_libraries(minigzip64 zlib)
        set_target_properties(minigzip64 PROPERTIES COMPILE_FLAGS "-D_FILE_OFFSET_BITS=64")
    endif()
endif()
