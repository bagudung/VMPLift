cmake_minimum_required(VERSION 3.20)
project(vmp-lift LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release)
endif()

# Static MSVC runtime (/MT) so the final exe doesn't need vcruntime/msvcp DLLs.
# Must be combined with the x64-windows-static vcpkg triplet so Capstone and
# Unicorn are also built as static libs (see the workflow's VCPKG_TARGET_TRIPLET).
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

find_package(capstone CONFIG REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)

# Prefer the vcpkg-provided CMake config for Unicorn Engine (recent Unicorn2
# ports ship one). Fall back to a manual find_path/find_library only if no
# config is found, since that manual path can silently pick up a wrong lib.
find_package(unicorn CONFIG QUIET)

if(TARGET unicorn::unicorn)
  set(UNICORN_TARGET unicorn::unicorn)
  message(STATUS "Unicorn Engine: using imported target unicorn::unicorn (CONFIG mode)")
elseif(TARGET unicorn)
  set(UNICORN_TARGET unicorn)
  message(STATUS "Unicorn Engine: using imported target unicorn (CONFIG mode)")
else()
  find_path(UNICORN_INCLUDE_DIR NAMES unicorn/unicorn.h)
  find_library(UNICORN_LIBRARY NAMES unicorn libunicorn)

  if(NOT UNICORN_INCLUDE_DIR OR NOT UNICORN_LIBRARY)
    message(FATAL_ERROR "Unicorn Engine not found (unicorn/unicorn.h or the unicorn library). Make sure vcpkg installed the 'unicorn' port.")
  endif()

  message(STATUS "Unicorn Engine: no CMake config found, using manual paths")
  message(STATUS "  include: ${UNICORN_INCLUDE_DIR}")
  message(STATUS "  library: ${UNICORN_LIBRARY}")

  add_library(unicorn::manual UNKNOWN IMPORTED)
  set_target_properties(unicorn::manual PROPERTIES
    IMPORTED_LOCATION "${UNICORN_LIBRARY}"
    INTERFACE_INCLUDE_DIRECTORIES "${UNICORN_INCLUDE_DIR}"
  )
  set(UNICORN_TARGET unicorn::manual)
endif()

file(GLOB VMPLIFT_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
list(REMOVE_ITEM VMPLIFT_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/main.cpp)

add_executable(vmp-lift main.cpp ${VMPLIFT_SOURCES})

target_link_libraries(vmp-lift PRIVATE
  capstone::capstone
  nlohmann_json::nlohmann_json
  ${UNICORN_TARGET}
)

if(MSVC)
  target_compile_options(vmp-lift PRIVATE /utf-8 /W3)
  target_compile_definitions(vmp-lift PRIVATE NOMINMAX WIN32_LEAN_AND_MEAN)
endif()
