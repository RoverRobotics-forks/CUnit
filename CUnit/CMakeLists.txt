cmake_minimum_required (VERSION 3.2)
Project(CUnit
	VERSION 3.1.2
	LANGUAGES C
	)

set(CUNIT_RELEASE cunity)

if(CMAKE_COMPILER_IS_GNUCXX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror -Werror=strict-prototypes")
endif(CMAKE_COMPILER_IS_GNUCXX)

if (MSVC)
  # force assert() to work on release mode windows
  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /UNDEBUG")
  message(STATUS "CMAKE_C_FLAGS_RELEASE=${CMAKE_C_FLAGS_RELEASE}")
endif()

set(CUNIT_VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}")
set(CUNIT_RELEASE "${PROJECT_VERSION_PATCH}")
configure_file(CUnit/CUnit.h.in ${CMAKE_CURRENT_LIST_DIR}/CUnit/CUnit.h @ONLY)


# Package Generator  #######################################################
# set(CPACK_PACKAGE_NAME "cunit")
set(CPACK_PACKAGE_DESCRIPTION "CUnit")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Dan's fork of the CUnit unit test framework")
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_PACKAGE_VENDOR "Digi Labs")
set(CPACK_PACKAGE_CONTACT "dan@digilabs.io")
set(CPACK_PACKAGE_FILE_NAME "cunit-${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/../COPYING)
include (CPack)

FILE(GLOB Basic Sources/Basic/*.c)
FILE(GLOB Automated Sources/Automated/*.c)
FILE(GLOB Framework Sources/Framework/*.c)
FILE(GLOB Console Sources/Console/*.c)
FILE(GLOB Test Sources/Test/*.c)
FILE(GLOB CI Sources/CI/*.c)

set(cunit_srcs
  ${Basic}
  ${Automated}
  ${CI}
  ${Framework}
  ${Console}
  )

set(cunit_libs "")

if(UNIX AND NOT APPLE AND NOT CMAKE_SYSTEM_NAME MATCHES "Linux")
  list(APPEND cunit_libs m)
endif()

set(main_target cunit)
add_library(${main_target} STATIC ${cunit_srcs})
target_link_libraries(${main_target}
  PUBLIC
    ${cunit_libs})
target_include_directories(${main_target}
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}>
    $<INSTALL_INTERFACE:include>)
FILE(GLOB cunit_public_headers "${CMAKE_CURRENT_SOURCE_DIR}/CUnit/*.h")
set_target_properties(${main_target} PROPERTIES PUBLIC_HEADER "${cunit_public_headers}")


# for internal test program
add_executable(cunit_test
  ${cunit_srcs}
  ${Test}
  )

add_test(NAME 
  cunit_test COMMAND $<TARGET_FILE:cunit_test>)

target_include_directories(cunit_test
  PRIVATE 
    Sources/Framework
    Sources/Test
    ${CMAKE_CURRENT_LIST_DIR}
  )
target_link_libraries(cunit_test
  ${cunit_libs})
target_compile_definitions(cunit_test
  PRIVATE
    CUNIT_BUILD_TESTS
    MEMTRACE
    CUNIT_DO_NOT_DEFINE_UNLESS_BUILDING_TESTS)

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

set(CMAKE_MODULES_DIR "${CMAKE_INSTALL_LIBDIR}/CUnit/cmake")

configure_package_config_file(CUnitConfig.cmake.in
  CUnitConfig.cmake
  INSTALL_DESTINATION "${CMAKE_MODULES_DIR}"
)
write_basic_package_version_file(
  CUnitConfigVersion.cmake
  COMPATIBILITY SameMajorVersion
)
install(
  TARGETS ${main_target} cunit_test
  EXPORT ${PROJECT_NAME}Targets
  ARCHIVE
  RUNTIME
  PUBLIC_HEADER DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/CUnit"
  )
install(EXPORT ${PROJECT_NAME}Targets
  DESTINATION ${CMAKE_MODULES_DIR}
  FILE ${PROJECT_NAME}Targets.cmake
  )
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/CUnitConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/CUnitConfigVersion.cmake
  DESTINATION ${CMAKE_MODULES_DIR}
  )

