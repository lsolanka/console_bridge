cmake_minimum_required(VERSION 2.8.6)
project(console_bridge)

set (CONSOLE_BRIDGE_MAJOR_VERSION 0)
set (CONSOLE_BRIDGE_MINOR_VERSION 3)
set (CONSOLE_BRIDGE_PATCH_VERSION 0)

set (CONSOLE_BRIDGE_VERSION ${CONSOLE_BRIDGE_MAJOR_VERSION}.${CONSOLE_BRIDGE_MINOR_VERSION}.${CONSOLE_BRIDGE_PATCH_VERSION})
message (STATUS "${PROJECT_NAME} version ${CONSOLE_BRIDGE_VERSION}")

include(GNUInstallDirs)
include(GenerateExportHeader)

# If compiler support symbol visibility, enable it.
include(CheckCCompilerFlag)
check_c_compiler_flag(-fvisibility=hidden HAS_VISIBILITY)
if (HAS_VISIBILITY)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")
endif()

if(MSVC OR MSVC90 OR MSVC10)
  set(MSVC ON)
endif (MSVC OR MSVC90 OR MSVC10)

include_directories(include ${PROJECT_BINARY_DIR})

add_library(${PROJECT_NAME} src/console.cpp)
target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include/>
)
generate_export_header(${PROJECT_NAME}
    EXPORT_MACRO_NAME CONSOLE_BRIDGE_DLLAPI)

install(TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}-targets 
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(DIRECTORY include/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  FILES_MATCHING PATTERN "*.h")

install(FILES ${PROJECT_BINARY_DIR}/${PROJECT_NAME}_export.h
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

if(WIN32 AND NOT CYGWIN)
  set(CMAKE_CONFIG_INSTALL_DIR CMake)
else()
  set(CMAKE_CONFIG_INSTALL_DIR ${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}/cmake/)
endif()

include(CMakePackageConfigHelpers)
set(PKG_NAME ${PROJECT_NAME})
set(PKG_LIBRARIES ${PROJECT_NAME})
set(cmake_conf_file "${PROJECT_NAME}-config.cmake")
configure_package_config_file("${cmake_conf_file}.in" "${CMAKE_BINARY_DIR}/${cmake_conf_file}"
                              INSTALL_DESTINATION ${CMAKE_CONFIG_INSTALL_DIR}
                              PATH_VARS CMAKE_INSTALL_FULL_INCLUDEDIR
                              NO_SET_AND_CHECK_MACRO
                              NO_CHECK_REQUIRED_COMPONENTS_MACRO)
set(cmake_conf_version_file "${PROJECT_NAME}-config-version.cmake")
# Use write_basic_package_version_file to generate a ConfigVersion file that
# allow users of gazebo to specify the API or version to depend on
# TODO: keep this instruction until deprecate Ubuntu/Precise and update with
# https://github.com/Kitware/CMake/blob/v2.8.8/Modules/CMakePackageConfigHelpers.cmake
include(WriteBasicConfigVersionFile)
write_basic_config_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${cmake_conf_version_file}
  VERSION "${CONSOLE_BRIDGE_VERSION}"
  COMPATIBILITY SameMajorVersion)

if(WIN32 AND NOT CYGWIN)
  set(CMAKE_CONFIG_INSTALL_DIR CMake)
else()
  set(CMAKE_CONFIG_INSTALL_DIR ${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}/cmake/)
endif()
install(EXPORT ${PROJECT_NAME}-targets
  FILE ${PROJECT_NAME}-targets.cmake
  NAMESPACE ros::
  DESTINATION ${CMAKE_CONFIG_INSTALL_DIR})
install(FILES
        "${CMAKE_BINARY_DIR}/${cmake_conf_file}"
        "${CMAKE_BINARY_DIR}/${cmake_conf_version_file}"
  DESTINATION ${CMAKE_CONFIG_INSTALL_DIR} COMPONENT cmake)

enable_testing()
add_subdirectory(test)
