cmake_minimum_required (VERSION 2.6.2)
project (cereal)

option(ENABLE_BOOST "Search and use boost" ON)
option(ENABLE_DOXYGEN "Search and use doxygen" ON)
option(SKIP_PORTABILITY_TEST "Skip portability tests" OFF)

set(CMAKE_CXX_FLAGS "-std=c++11 -Wall -Werror -g -Wextra -Wshadow -pedantic ${CMAKE_CXX_FLAGS}")

include_directories(./include)

if(ENABLE_BOOST)
    find_package(Boost COMPONENTS serialization unit_test_framework)

    if(Boost_FOUND)
        include_directories(${Boost_INCLUDE_DIRS})
        enable_testing()
        add_subdirectory(unittests)
    endif(Boost_FOUND)
endif(ENABLE_BOOST)

add_subdirectory(sandbox)

if(ENABLE_DOXYGEN)
    find_package(Doxygen)
    if(DOXYGEN_FOUND)

        configure_file("${CMAKE_CURRENT_SOURCE_DIR}/doc/doxygen.in" "${CMAKE_CURRENT_BINARY_DIR}/doxygen.cfg" @ONLY)
        add_custom_target(doc
            COMMAND ${DOXYGEN_EXECUTABLE} "${CMAKE_CURRENT_BINARY_DIR}/doxygen.cfg"
            WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
            COMMENT "Generating API documentation with Doxygen" VERBATIM
            )

        configure_file("${CMAKE_CURRENT_SOURCE_DIR}/scripts/updatedoc.in" "${CMAKE_CURRENT_BINARY_DIR}/updatedoc.sh" @ONLY)
        add_custom_target(update-doc
            COMMAND "${CMAKE_CURRENT_BINARY_DIR}/updatedoc.sh"
            DEPENDS doc
            COMMENT "Copying documentation to gh-pages branch" VERBATIM
            )

    endif(DOXYGEN_FOUND)
endif(ENABLE_DOXYGEN)

find_package(Git)
#packaging
file(GLOB CEREAL_HEADERS "include/cereal/*.hpp")
file(GLOB CEREAL_HEADERS_ARCHIVES "include/cereal/archives/*.hpp")
file(GLOB CEREAL_HEADERS_TYPES "include/cereal/types/*.hpp")
file(GLOB CEREAL_HEADERS_DETAILS "include/cereal/details/*.hpp")
file(GLOB CEREAL_HEADERS_EXTERNAL "include/cereal/external/*.hpp")
file(GLOB EXTERNAL_RAPIDJSON_HEADERS "include/cereal/external/rapidjson/*.h")
file(GLOB EXTERNAL_RAPIDJSON_LICENSE "include/cereal/external/rapidjson/license.txt")
file(GLOB EXTERNAL_RAPIDJSON_INTERNAL_HEADERS "include/cereal/external/rapidjson/internal/*.h")
file(GLOB EXTERNAL_RAPIDXML_HEADERS "include/cereal/external/rapidxml/*.hpp")
file(GLOB EXTERNAL_RAPIDXML_MANUAL "include/cereal/external/rapidxml/manual.html")
file(GLOB EXTERNAL_RAPIDXML_LICENSE "include/cereal/external/rapidxml/license.txt")
install (FILES ${CEREAL_HEADERS} DESTINATION include/cereal)
install (FILES ${CEREAL_HEADERS_ARCHIVES} DESTINATION include/cereal/archives)
install (FILES ${CEREAL_HEADERS_TYPES} DESTINATION include/cereal/types)
install (FILES ${CEREAL_HEADERS_DETAILS} DESTINATION include/cereal/details)
install (FILES ${CEREAL_HEADERS_EXTERNAL} DESTINATION include/cereal/external)
install (FILES ${EXTERNAL_RAPIDJSON_HEADERS} DESTINATION include/cereal/external/rapidjson)
install (FILES ${EXTERNAL_RAPIDJSON_LICENSE} DESTINATION include/cereal/external/rapidjson)
install (FILES ${EXTERNAL_RAPIDJSON_INTERNAL_HEADERS} DESTINATION include/cereal/external/rapidjson/internal)
install (FILES ${EXTERNAL_RAPIDXML_HEADERS} DESTINATION include/cereal/external/rapidxml)
install (FILES ${EXTERNAL_RAPIDXML_MANUAL} DESTINATION include/cereal/external/rapidxml)
install (FILES ${EXTERNAL_RAPIDXML_LICENSE} DESTINATION include/cereal/external/rapidxml)

if(GIT_FOUND)
    execute_process(COMMAND ${GIT_EXECUTABLE} describe --abbrev=0 --tags
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        OUTPUT_VARIABLE VERSION_STRING
        OUTPUT_STRIP_TRAILING_WHITESPACE)

    if(VERSION_STRING)
        set(CPACK_PACKAGE_VERSION ${VERSION_STRING})
    else()
        set(CPACK_PACKAGE_VERSION "v1.1.2")
    endif()
else()
    set(CPACK_PACKAGE_VERSION "v1.1.2")
endif()

set(BUILD_NUM_FILE "${CMAKE_SOURCE_DIR}/build.num")
set(VERSION_FILE "${CMAKE_SOURCE_DIR}/build.version")
if (EXISTS ${BUILD_NUM_FILE})
    file(READ ${BUILD_NUM_FILE} BUILD_NUM)
else()
    set(BUILD_NUM 1)
endif()

if (EXISTS ${VERSION_FILE})
    file(READ ${VERSION_FILE} LAST_VERSION)
else()
    set(LAST_VERSION ${CPACK_PACKAGE_VERSION})
    file(WRITE ${VERSION_FILE} ${LAST_VERSION})
endif()

if (${LAST_VERSION} STREQUAL ${CPACK_PACKAGE_VERSION})
    math(EXPR BUILD_NUM ${BUILD_NUM}+1)
else()
    set(BUILD_NUM 1)
    set(LAST_VERSION ${CPACK_PACKAGE_VERSION})
    file(WRITE ${VERSION_FILE} ${LAST_VERSION})
endif()
file(WRITE ${BUILD_NUM_FILE} ${BUILD_NUM})

message(STATUS "Build num = " ${BUILD_NUM})
message(STATUS "Current Version = " ${CPACK_PACKAGE_VERSION})

set(CPACK_GENERATOR "RPM")
set(CPACK_PACKAGE_NAME "cereal")
set(CPACK_PACKAGE_RELEASE ${BUILD_NUM})
set(CPACK_PACKAGE_CONTACT "cereal")
set(CPACK_PACKAGE_VENDOR "http://uscilab.github.io/cereal/")
set(CPACK_PACKAGING_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX})
set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-${CPACK_PACKAGE_RELEASE}.${CMAKE_SYSTEM_PROCESSOR}")
set(CPACK_RPM_EXCLUDE_FROM_AUTO_FILELIST_ADDITION "/usr/share/cmake")
set(CPACK_RPM_PACKAGE_RELEASE ${BUILD_NUM})
set(CPACK_RPM_PACKAGE_SUMMARY "A C++11 library for serialization")
set(CPACK_RPM_PACKAGE_LICENSE "BSD")
set(CPACK_RPM_PACKAGE_GROUP "Development/Libraries")
set(CPACK_RPM_PACKAGE_DESCRIPTION "cereal is a header-only C++11 serialization library. cereal takes arbitrary
data types and reversibly turns them into different representations, such as
compact binary encodings, XML, or JSON. cereal was designed to be fast,
light-weight, and easy to extend - it has no external dependencies and can be
easily bundled with other code or used standalone.")
include(CPack)
