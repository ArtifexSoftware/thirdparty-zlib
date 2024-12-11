add_test(NAME zlib_install
    COMMAND ${CMAKE_COMMAND} --install ${zlib_BINARY_DIR}
        --prefix ${CMAKE_CURRENT_BINARY_DIR}/test_install
        --config $<CONFIG>
    WORKING_DIRECTORY ${zlib_BINARY_DIR})

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
add_test(NAME zlib_configure_find_package
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

add_test(NAME zlib_build_find_package
    COMMAND ${CMAKE_COMMAND} --build .
    --config $<CONFIG>
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/findpackage_test_build)

add_test(NAME zlib_test_find_package
    COMMAND ${CMAKE_CTEST_COMMAND} -C $<CONFIG>
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/findpackage_test_build)

set_tests_properties(zlib_test_install PROPERTIES
    FIXTURES_SETUP install)

set_tests_properties(zlib_test_configure_find_package PROPERTIES
    FIXTURES_REQUIRED install
    FIXTURES_SETUP fp_config)

set_tests_properties(zlib_test_build_find_package PROPERTIES
    FIXTURES_REQUIRED fp_config
    FIXTURES_SETUP fp_build)

set_tests_properties(zlib_test_test_find_package PROPERTIES
    FIXTURES_REQUIRED fp_build
    ENVIRONMENT CTEST_OUTPUT_ON_FAILURE=1)

#
# add_subdirectory_test
#
add_test(NAME zlib_configure_add_subdirectory
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

add_test(NAME zlib_build_add_subdirectory
    COMMAND ${CMAKE_COMMAND} --build .
    --config $<CONFIG>
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/add_subdirectory_test_build)

add_test(NAME zlib_test_add_subdirectory
    COMMAND ${CMAKE_CTEST_COMMAND} -C $<CONFIG>
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/add_subdirectory_test_build)

set_tests_properties(zlib_test_configure_add_subdirectory PROPERTIES
    FIXTURES_SETUP as_config)

set_tests_properties(zlib_test_build_add_subdirectory PROPERTIES
    FIXTURES_REQUIRED as_config
    FIXTURES_SETUP as_build)

set_tests_properties(zlib_test_test_add_subdirectory PROPERTIES
    FIXTURES_REQUIRED as_build
    ENVIRONMENT CTEST_OUTPUT_ON_FAILURE=1)

#
# add_subdirectory_exclude_test
#
add_test(NAME zlib_configure_add_subdirectory_exclude
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

add_test(NAME zlib_build_add_subdirectory_exclude
    COMMAND ${CMAKE_COMMAND} --build .
    --config $<CONFIG>
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/add_subdirectory_exclude_test_build)

add_test(NAME zlib_test_add_subdirectory_exclude
    COMMAND ${CMAKE_CTEST_COMMAND} -C $<CONFIG>
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/add_subdirectory_exclude_test_build)

set_tests_properties(zlib_test_configure_add_subdirectory_exclude PROPERTIES
    FIXTURES_SETUP asx_config)

set_tests_properties(zlib_test_build_add_subdirectory_exclude PROPERTIES
    FIXTURES_REQUIRED as_config
    FIXTURES_SETUP asx_build)

set_tests_properties(zlib_test_test_add_subdirectory_exclude PROPERTIES
    FIXTURES_REQUIRED asx_build
    ENVIRONMENT CTEST_OUTPUT_ON_FAILURE=1)
