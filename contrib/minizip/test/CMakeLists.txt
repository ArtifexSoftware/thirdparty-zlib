add_test(NAME minizip_test_install
    COMMAND ${CMAKE_COMMAND} --install ${zlib_BINARY_DIR}
        --prefix ${CMAKE_CURRENT_BINARY_DIR}/test_install
        --config $<CONFIG>
    WORKING_DIRECTORY ${zlib_BINARY_DIR})

set_tests_properties(minizip_test_install
    PROPERTIES
        FIXTURES_SETUP minizip_install)

file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/findpackage_test)
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/add_subdirectoy_test)
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/add_subdirectory_exclude_test)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/find_package_test.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/findpackage_test/CMakeLists.txt)

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/add_subdirectory_test.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/add_subdirectoy_test/CMakeLists.txt)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/add_subdirectory_exclude_test.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/add_subdirectory_exclude_test/CMakeLists.txt)

# CMAKE_GENERATOR_PLATFORM doesn't work in the if
set(GENERATOR ${CMAKE_GENERATOR_PLATFORM})

if(GENERATOR)
    set(PLATFORM "-A ${GENERATOR}")
endif(GENERATOR)

#
# findpackage_test
#
add_test(NAME minizip_configure_find_package
    COMMAND ${CMAKE_COMMAND}
        ${PLATFORM}
        -B${CMAKE_CURRENT_BINARY_DIR}/findpackage_test_build
        -DCMAKE_BUILD_TYPE=$<CONFIG>
        -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
        -DCMAKE_C_FLAGS=${CMAKE_C_FLAGS}
        -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/test_install
        -DZDIR=${CMAKE_CURRENT_BINARY_DIR}/test_install/${CMAKE_INSTALL_LIBDIR}
        --fresh
        -G "${CMAKE_GENERATOR}"
        -S${CMAKE_CURRENT_BINARY_DIR}/findpackage_test)

add_test(NAME minizip_build_find_package
    COMMAND ${CMAKE_COMMAND} --build .
    --config $<CONFIG>
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/findpackage_test_build)

add_test(NAME minizip_test_find_package
    COMMAND ${CMAKE_CTEST_COMMAND} -C $<CONFIG>
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/findpackage_test_build)

set_tests_properties(minizip_configure_find_package PROPERTIES
    FIXTURES_REQUIRED minizip_install
    FIXTURES_SETUP mzfp_config)

set_tests_properties(minizip_build_find_package PROPERTIES
    FIXTURES_REQUIRED mzfp_config
    FIXTURES_SETUP mzfp_build)

set_tests_properties(minizip_test_find_package PROPERTIES
    FIXTURES_REQUIRED mzfp_build
    ENVIRONMENT CTEST_OUTPUT_ON_FAILURE=1)

#
# add_subdirectory_test
#
add_test(NAME minizip_configure_add_subdirectory
    COMMAND ${CMAKE_COMMAND}
    ${PLATFORM}
    -B${CMAKE_CURRENT_BINARY_DIR}/add_subdirectory_test_build
    -DCMAKE_BUILD_TYPE=$<CONFIG>
    -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
    -DCMAKE_C_FLAGS=${CMAKE_C_FLAGS}
    -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/test_install
    --fresh
    -G "${CMAKE_GENERATOR}"
    -S${CMAKE_CURRENT_BINARY_DIR}/findpackage_test)

add_test(NAME minizip_build_add_subdirectory
    COMMAND ${CMAKE_COMMAND} --build .
    --config $<CONFIG>
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/add_subdirectory_test_build)

add_test(NAME minizip_test_add_subdirectory
    COMMAND ${CMAKE_CTEST_COMMAND} -C $<CONFIG>
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/add_subdirectory_test_build)

set_tests_properties(minizip_configure_add_subdirectory PROPERTIES
    FIXTURES_REQUIRED minizip_install
    FIXTURES_SETUP mzas_config)

set_tests_properties(minizip_build_add_subdirectory PROPERTIES
    FIXTURES_REQUIRED mzas_config
    FIXTURES_SETUP mzas_build)

set_tests_properties(minizip_test_add_subdirectory PROPERTIES
    FIXTURES_REQUIRED mzas_build
    ENVIRONMENT CTEST_OUTPUT_ON_FAILURE=1)

#
# add_subdirectory_exclude_test
#
add_test(NAME minizip_configure_add_subdirectory_exclude
    COMMAND ${CMAKE_COMMAND}
    ${PLATFORM}
    -B${CMAKE_CURRENT_BINARY_DIR}/add_subdirectory_exclude_test_build
    -DCMAKE_BUILD_TYPE=$<CONFIG>
    -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
    -DCMAKE_C_FLAGS=${CMAKE_C_FLAGS}
    -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/test_install
    --fresh
    -G "${CMAKE_GENERATOR}"
    -S${CMAKE_CURRENT_BINARY_DIR}/findpackage_test)

add_test(NAME minizip_build_add_subdirectory_exclude
    COMMAND ${CMAKE_COMMAND} --build .
    --config $<CONFIG>
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/add_subdirectory_exclude_test_build)

add_test(NAME minizip_test_add_subdirectory_exclude
    COMMAND ${CMAKE_CTEST_COMMAND} -C $<CONFIG>
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/add_subdirectory_exclude_test_build)

set_tests_properties(minizip_configure_add_subdirectory_exclude PROPERTIES
    FIXTURES_REQUIRED minizip_install
    FIXTURES_SETUP mzasx_config)

set_tests_properties(minizip_build_add_subdirectory_exclude PROPERTIES
    FIXTURES_REQUIRED mzasx_config
    FIXTURES_SETUP mzasx_build)

set_tests_properties(minizip_test_add_subdirectory_exclude PROPERTIES
    FIXTURES_REQUIRED mzasx_build
    ENVIRONMENT CTEST_OUTPUT_ON_FAILURE=1)
