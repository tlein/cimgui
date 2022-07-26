cmake_minimum_required(VERSION 3.15)

include(CMakePackageConfigHelpers)

project(cimgui)

set(cimgui_VERSION 1.88)

set (CMAKE_CXX_STANDARD 11)

if (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_tables.cpp)
	set(TABLES_SOURCE "imgui/imgui_tables.cpp")
else()
	set(TABLES_SOURCE "")
endif()


#general settings
file(GLOB IMGUI_SOURCES
    cimgui.cpp
    imgui/imgui.cpp
    imgui/imgui_draw.cpp
    imgui/imgui_demo.cpp
    imgui/imgui_widgets.cpp
    imgui/backends/imgui_impl_sdl.cpp
    imgui/backends/imgui_impl_sdlrenderer.cpp
	${TABLES_SOURCE}
)

find_package(SDL2 2.0.022 CONFIG REQUIRED)

set(IMGUI_STATIC "no" CACHE STRING "Build as a static library")
set(IMGUI_FREETYPE "no" CACHE STRING "Build with freetype library")
set(IMGUI_LIBRARIES SDL2::SDL2)

if(IMGUI_FREETYPE)
	FIND_PACKAGE(freetype REQUIRED PATHS ${FREETYPE_PATH})
	list(APPEND IMGUI_LIBRARIES freetype)
	list(APPEND IMGUI_SOURCES imgui/misc/freetype/imgui_freetype.cpp)
	add_definitions("-DCIMGUI_FREETYPE=1")
endif(IMGUI_FREETYPE)

#add library and link
if (IMGUI_STATIC)
    add_library(cimgui STATIC ${IMGUI_SOURCES})
else (IMGUI_STATIC)
    add_library(cimgui SHARED ${IMGUI_SOURCES})
endif (IMGUI_STATIC)
add_library(cimgui::cimgui ALIAS cimgui)

target_compile_definitions(cimgui PUBLIC IMGUI_DISABLE_OBSOLETE_FUNCTIONS=1)
if (WIN32)
    target_compile_definitions(cimgui PUBLIC IMGUI_IMPL_API=extern\t\"C\"\t__declspec\(dllexport\))
    list(APPEND IMGUI_LIBRARIES imm32)
else (WIN32)
    target_compile_definitions(cimgui PUBLIC IMGUI_IMPL_API=extern\t\"C\"\t)
endif (WIN32)

target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/imgui)
set_target_properties(
  cimgui
    PROPERTIES
      DEBUG_POSTFIX "-d"
)
target_link_libraries(cimgui ${IMGUI_LIBRARIES})

# Enumerate headers
set(
  cimgui_PUBLIC_HEADERS
    "cimgui.h"
    "generator/output/cimgui_impl.h"
)

set_target_properties(
  cimgui
    PROPERTIES 
      PUBLIC_HEADER "${cimgui_PUBLIC_HEADERS}"
)

# Configure config package file pased on .in file
configure_package_config_file(
  cimguiConfig.cmake.in
  ${CMAKE_BINARY_DIR}/cmake/cimgui/cimguiConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake
)

write_basic_package_version_file(
  ${CMAKE_BINARY_DIR}/cmake/cimgui/cimguiConfigVersion.cmake
  VERSION ${cimgui_VERSION}
  COMPATIBILITY SameMajorVersion
)

# install library targets
install(
  TARGETS cimgui
  EXPORT cimguiTargets
  LIBRARY 
    DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE
    DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME
    DESTINATION ${CMAKE_INSTALL_BINDIR}
  PUBLIC_HEADER
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/cimgui
)

# install the cimguiConfig.cmake and cimguiConfigVersion.cmake
install(
  FILES
    ${CMAKE_BINARY_DIR}/cmake/cimgui/cimguiConfig.cmake
    ${CMAKE_BINARY_DIR}/cmake/cimgui/cimguiConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/cimgui
  COMPONENT cimgui_DEVELOPMENT
)

# install the cimguiTargets.cmake file
install(
  EXPORT cimguiTargets
  FILE cimguiTargets.cmake
  NAMESPACE cimgui::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/cimgui
  COMPONENT devel
)


#test
set(CIMGUI_TEST "no" CACHE STRING "Enable compilation of a test unit based on imgui null")

if (CIMGUI_TEST)
  add_subdirectory(test)
endif ()
