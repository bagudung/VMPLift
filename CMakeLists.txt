cmake_minimum_required(VERSION 3.20)
project(vmp-lift LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release)
endif()

find_package(capstone CONFIG REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)

# vcpkg's "unicorn" port does not reliably ship a CMake config file across
# versions, so locate it the plain way instead of find_package(... CONFIG).
find_path(UNICORN_INCLUDE_DIR NAMES unicorn/unicorn.h)
find_library(UNICORN_LIBRARY NAMES unicorn unicorn.dll libunicorn)

if(NOT UNICORN_INCLUDE_DIR OR NOT UNICORN_LIBRARY)
  message(FATAL_ERROR "Unicorn Engine not found (unicorn/unicorn.h or the unicorn library). Make sure vcpkg installed the 'unicorn' port.")
endif()

file(GLOB VMPLIFT_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
list(REMOVE_ITEM VMPLIFT_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/main.cpp)

add_executable(vmp-lift main.cpp ${VMPLIFT_SOURCES})

target_include_directories(vmp-lift PRIVATE ${UNICORN_INCLUDE_DIR})

target_link_libraries(vmp-lift PRIVATE
  capstone::capstone
  nlohmann_json::nlohmann_json
  ${UNICORN_LIBRARY}
)

if(MSVC)
  target_compile_options(vmp-lift PRIVATE /utf-8 /W3)
  target_compile_definitions(vmp-lift PRIVATE NOMINMAX WIN32_LEAN_AND_MEAN)
endif()
