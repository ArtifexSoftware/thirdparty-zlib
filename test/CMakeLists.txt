add_test(NAME zlib_test_install
    COMMAND ${CMAKE_COMMAND} --install ${zlib_BINARY_DIR}
        --prefix ${CMAKE_CURRENT_BINARY_DIR}/test_install
        --config $<CONFIG>
    WORKING_DIRECTORY ${zlib_BINARY_DIR})

file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/findpackage_test)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/find_package_test.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/findpackage_test/CMakeLists.txt)

add_test(NAME zlib_test_configure_find_package
   COMMAND ${CMAKE_COMMAND}
       -B${CMAKE_CURRENT_BINARY_DIR}/findpackage_test_build
       -DCMAKE_BUILD_TYPE=$<CONFIG>
       -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/test_install
       -DZDIR=${CMAKE_CURRENT_BINARY_DIR}/test_install/${CMAKE_INSTALL_LIBDIR}
       --fresh
       -G "${CMAKE_GENERATOR}"
       -S${CMAKE_CURRENT_BINARY_DIR}/findpackage_test)


add_test(NAME zlib_test_build_find_package
    COMMAND ${CMAKE_COMMAND} --build
            ${CMAKE_CURRENT_BINARY_DIR}/findpackage_test_build
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/findpackage_test_build)

add_test(NAME zlib_test_test_find_package
    COMMAND ${CMAKE_CTEST_COMMAND} -C $<CONFIG>
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/findpackage_test_build)

set_tests_properties(zlib_test_install PROPERTIES
    FIXTURES_SETUP install)

set_tests_properties(zlib_test_configure_find_package PROPERTIES
    FIXTURES_REQUIRED install
    FIXTURES_SETUP config)

set_tests_properties(zlib_test_build_find_package PROPERTIES
    FIXTURES_REQUIRED config
    FIXTURES_SETUP build)

set_tests_properties(zlib_test_test_find_package PROPERTIES
    FIXTURES_REQUIRED build)
