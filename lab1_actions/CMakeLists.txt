cmake_minimum_required(VERSION 3.10)
cmake_policy(SET CMP0167 OLD)

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VERSION 0.0.${PATCH_VERSION})

project(helloworld VERSION ${PROJECT_VERSION})

find_package(Boost REQUIRED COMPONENTS unit_test_framework)

configure_file(version.h.in version.h)

add_library(helloworld lib.cpp)
add_executable(helloworld_cli main.cpp)
add_executable(test_version test_version.cpp)

set_target_properties(helloworld helloworld_cli test_version PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)

# Динамическая линковка для Boost тестов
target_compile_definitions(test_version PRIVATE BOOST_TEST_DYN_LINK)

target_include_directories(helloworld PRIVATE "${CMAKE_BINARY_DIR}")
target_include_directories(helloworld_cli PRIVATE "${CMAKE_BINARY_DIR}")
target_include_directories(test_version PRIVATE "${CMAKE_BINARY_DIR}" ${Boost_INCLUDE_DIR})

target_link_libraries(helloworld_cli PRIVATE helloworld)
target_link_libraries(test_version PRIVATE helloworld ${Boost_LIBRARIES})

if(MSVC)
    target_compile_options(helloworld PRIVATE /W4)
    target_compile_options(helloworld_cli PRIVATE /W4)
    target_compile_options(test_version PRIVATE /W4)
else()
    target_compile_options(helloworld PRIVATE -Wall -Wextra -pedantic -Werror)
    target_compile_options(helloworld_cli PRIVATE -Wall -Wextra -pedantic -Werror)
    target_compile_options(test_version PRIVATE -Wall -Wextra -pedantic -Werror)
endif()

install(TARGETS helloworld_cli RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)
set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set(CPACK_PACKAGE_CONTACT example@example.com)
include(CPack)

enable_testing()
add_test(NAME test_version COMMAND test_version)
