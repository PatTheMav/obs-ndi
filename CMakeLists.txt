cmake_minimum_required(VERSION 3.16)

project(obs-ndi VERSION 5.0.0)
add_library(obs-ndi MODULE)

set(MACOS_BUNDLEID "fr.palakis.obs-ndi")

find_package(libobs REQUIRED)
find_package(obs-frontend-api REQUIRED)
include(external/ObsPluginHelpers.cmake)
find_package(
  Qt5
  COMPONENTS Core Widgets
  REQUIRED)

if("${CMAKE_BINARY_DIR}" STREQUAL "${CMAKE_SOURCE_DIR}")
  message(
    FATAL_ERROR
      "OBS: You cannot build in a source directory (or any directory with "
      "CMakeLists.txt file). Please make a build subdirectory. Feel free to "
      "remove CMakeCache.txt and CMakeFiles.")
endif()

target_include_directories(
  obs-ndi PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/lib/ndi
                  ${CMAKE_BINARY_DIR}/config)

target_link_libraries(obs-ndi PRIVATE OBS::libobs OBS::obs-frontend-api
                                      Qt5::Core Qt5::Widgets)

set_target_properties(
  obs-ndi
  PROPERTIES AUTOMOC ON
             AUTOUIC ON
             AUTORCC ON)

target_compile_features(obs-ndi PRIVATE cxx_std_17)

set(NDILIB_HEADERS_VERSION 5.0.0)
configure_file(src/obs-ndi-macros.h.in config/obs-ndi-macros.generated.h)

target_sources(
  obs-ndi
  PRIVATE src/obs-ndi.cpp
          src/obs-ndi.h
          src/obs-ndi-input.cpp
          src/obs-ndi-input.h
          src/obs-ndi-output.h
          src/obs-ndi-input-utils.cpp
          src/obs-ndi-output.cpp
          src/forms/SettingsDialog.cpp
          src/forms/SettingsDialog.h
          src/forms/resources.qrc
          config/obs-ndi-macros.generated.h)

if(OS_WINDOWS)
  if(MSVC)
    target_compile_options(obs-ndi PRIVATE /MP /d2FH4-)
  endif()
elseif(OS_MACOS)
  set(MACOSX_PLUGIN_GUI_IDENTIFIER "${MACOS_BUNDLEID}")
  set(MACOSX_PLUGIN_BUNDLE_VERSION "${CMAKE_PROJECT_VERSION}")
  set(MACOSX_PLUGIN_SHORT_VERSION_STRING "1")

  target_compile_options(
    obs-ndi PRIVATE -Wall -Wextra -Werror-implicit-function-declaration
                    -stdlib=libc++ -fvisibility=default)

  set_target_properties(obs-ndi PROPERTIES PREFIX "")
else()
  # ARM optimizations
  if(${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
    target_compile_options(obs-ndi -std=c++17 -mtune=arm7 -Ofast)
  else()
    target_compile_options(obs-ndi -std=c++17 -mtune=core2 -Ofast)
  endif()

  target_compile_options(obs-ndi PRIVATE -Wall -Wextra)

  set_target_properties(obs-ndi PROPERTIES PREFIX "")
endif()

setup_plugin_target(obs-ndi)
