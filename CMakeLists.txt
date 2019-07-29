cmake_minimum_required(VERSION 3.10.0)

project(restclient-cpp)

message(STATUS "Configured for: ${CMAKE_BUILD_TYPE}")
add_custom_target(BeginMessage ALL
    ${CMAKE_COMMAND} -E cmake_echo_color --white "Compiling for: $<CONFIG>"
    COMMENT "Compile Info" )

set(CMAKE_DEBUG_POSTFIX d)

set(CMAKE_CXX_STANDARD 11)

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/modules")


if(POLICY CMP0074)
  cmake_policy(SET CMP0074 NEW)
endif()

include(CheckIncludeFiles)

find_package(Threads REQUIRED)
#find_package(CURL REQUIRED PATHS ${CMAKE_INSTALL_PERFIX})

add_library(restclient-cpp SHARED
  source/restclient.cc
  source/connection.cc
  source/helpers.cc
)
set_property(TARGET restclient-cpp PROPERTY SOVERSION 2.1.1)

target_compile_features(restclient-cpp PUBLIC cxx_std_11)

list(APPEND restclient-cpp_PUBLIC_HEADERS
  include/restclient-cpp/restclient.h
  "${CMAKE_CURRENT_BINARY_DIR}/include/restclient-cpp/version.h"
  include/restclient-cpp/connection.h
  include/restclient-cpp/helpers.h
)
# target_sources(restclient-cpp PRIVATE ${restclient-cpp_PUBLIC_HEADERS})
set_property(TARGET restclient-cpp PROPERTY
  PUBLIC_HEADER ${restclient-cpp_PUBLIC_HEADERS})
target_include_directories(restclient-cpp PRIVATE include)
target_include_directories(restclient-cpp PRIVATE ${CMAKE_PREFIX_PATH}/include)
link_directories(${CMAKE_INSTALL_PREFIX}/lib)

if(NOT EXISTS "${CMAKE_CURRENT_BINARY_DIR}/version.h.in")
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/version.h.in" [=[
#ifndef INCLUDE_RESTCLIENT_CPP_VERSION_H_
#define INCLUDE_RESTCLIENT_CPP_VERSION_H_
#define RESTCLIENT_VERSION "@restclient-cpp_VERSION@"
#endif  // INCLUDE_RESTCLIENT_CPP_VERSION_H_
]=])
endif()
configure_file("${CMAKE_CURRENT_BINARY_DIR}/version.h.in"
  "${CMAKE_CURRENT_BINARY_DIR}/include/restclient-cpp/version.h")
target_include_directories(restclient-cpp PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/include")

if (APPLE)
target_link_libraries(restclient-cpp
  PUBLIC Threads::Threads curl
)
else(APPLE)
target_link_libraries(restclient-cpp
  PUBLIC Threads::Threads
)
endif(APPLE)

set(INCLUDE_INSTALL_DIR "include/restclient-cpp" )
set(CONFIG_INSTALL_DIR "lib/cmake/restclient-cpp" )
set(RUNTIME_INSTALL_DIR "bin" )
set(LIB_INSTALL_DIR "lib" )
set(DATA_INSTALL_DIR "share/restclient-cpp" )

install(TARGETS restclient-cpp EXPORT restclient-cppTargets
  PUBLIC_HEADER DESTINATION ${INCLUDE_INSTALL_DIR}
  RUNTIME DESTINATION ${RUNTIME_INSTALL_DIR}
  LIBRARY DESTINATION ${LIB_INSTALL_DIR}
  ARCHIVE DESTINATION ${LIB_INSTALL_DIR}
  RESOURCE DESTINATION ${DATA_INSTALL_DIR}
)

include(CMakePackageConfigHelpers)
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake.in"
  "@PACKAGE_INIT@\ninclude(\${CMAKE_CURRENT_LIST_DIR}/\@PROJECT_NAME\@Targets.cmake)\n")
configure_package_config_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION
    ${CMAKE_INSTALL_PREFIX}/${CONFIG_INSTALL_DIR}
  PATH_VARS
    INCLUDE_INSTALL_DIR
    CONFIG_INSTALL_DIR
    RUNTIME_INSTALL_DIR
    LIB_INSTALL_DIR
    DATA_INSTALL_DIR
)
  
#write_basic_package_version_file(
#  "${CMAKE_CURRENT_BINARY_DIR}/restclient-cppConfigVersion.cmake"
#  COMPATIBILITY
#    SameMajorVersion
#)

# export targets for find_package config mode
install(EXPORT restclient-cppTargets DESTINATION ${CONFIG_INSTALL_DIR})

install(
  FILES
    ${CMAKE_CURRENT_BINARY_DIR}/restclient-cppConfig.cmake
  DESTINATION
    ${CMAKE_INSTALL_PREFIX}/${CONFIG_INSTALL_DIR}
)





