cmake_minimum_required (VERSION 3.15)
project (neo_minideen CXX)
file(GLOB CODE "src/*.cpp")
file(GLOB SSE2_CODE_IMPL "src/*SSE2.cpp")
file(GLOB AVX2_CODE_IMPL "src/*AVX2.cpp")
add_library(neo-minideen SHARED main.cpp src/version.rc ${CODE} ${CODE_IMPL})
set_property(TARGET neo-minideen PROPERTY CXX_STANDARD 17)

find_package(Git REQUIRED)
execute_process(COMMAND ${GIT_EXECUTABLE} describe --first-parent --tags --always OUTPUT_VARIABLE GIT_REPO_VERSION)
string(REGEX REPLACE "(r[0-9]+).*" "\\1" VERSION ${GIT_REPO_VERSION})

configure_file (
  "${PROJECT_SOURCE_DIR}/src/version.hpp.in"
  "${PROJECT_SOURCE_DIR}/src/version.hpp"
)
configure_file (
  "${PROJECT_SOURCE_DIR}/src/version.rc.in"
  "${PROJECT_SOURCE_DIR}/src/version.rc"
)

find_package(PkgConfig REQUIRED)

pkg_check_modules(AVISYNTH avisynth)
if(AVISYNTH_FOUND)
  include_directories(${AVISYNTH_INCLUDE_DIRS})
else()
  include_directories(include/avisynth)
endif()

pkg_check_modules(VAPOURSYNTH vapoursynth)
if(VAPOURSYNTH_FOUND)
  include_directories(${VAPOURSYNTH_INCLUDE_DIRS})
else()
  include_directories(include/vapoursynth)
endif()

include_directories(.)
include_directories(include/dualsynth)

if(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
  set_source_files_properties(${SSE2_CODE_IMPL} PROPERTIES COMPILE_FLAGS "/arch:SSE2")
  set_source_files_properties(${AVX2_CODE_IMPL} PROPERTIES COMPILE_FLAGS "/arch:AVX2")

  if (CMAKE_GENERATOR_TOOLSET MATCHES "v[0-9]*_xp")
    target_compile_definitions(neo-minideen PRIVATE WINVER=0x502 _WIN32_WINNT=0x502)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Zc:threadSafeInit-")
  endif()

elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Intel")
  set_source_files_properties(${SSE2_CODE_IMPL} PROPERTIES COMPILE_FLAGS "/arch:SSE2")
  set_source_files_properties(${AVX2_CODE_IMPL} PROPERTIES COMPILE_FLAGS "/arch:AVX2")

else()
  set_source_files_properties(${SSE2_CODE_IMPL} PROPERTIES COMPILE_FLAGS "-msse2")
  set_source_files_properties(${AVX2_CODE_IMPL} PROPERTIES COMPILE_FLAGS "-mavx2")

endif()

add_custom_command(
  TARGET neo-minideen POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:neo-minideen> "../Release_${VERSION}/${_DIR}/$<TARGET_FILE_NAME:neo-minideen>"
)
