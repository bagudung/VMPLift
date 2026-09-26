cmake_minimum_required(VERSION 3.20)
project(vmp-lift LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release)
endif()

find_package(capstone CONFIG REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)

file(GLOB VMPLIFT_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
list(REMOVE_ITEM VMPLIFT_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/main.cpp)

add_executable(vmp-lift main.cpp ${VMPLIFT_SOURCES})

target_link_libraries(vmp-lift PRIVATE
  capstone::capstone
  nlohmann_json::nlohmann_json
)

if(MSVC)
  target_compile_options(vmp-lift PRIVATE /utf-8 /W3)
  target_compile_definitions(vmp-lift PRIVATE NOMINMAX WIN32_LEAN_AND_MEAN)
endif()
