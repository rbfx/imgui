cmake_minimum_required(VERSION 3.28)

project(imgui)

add_library(imgui STATIC
    imgui.cpp
    imgui_widgets.cpp
    imgui_tables.cpp
    imgui_draw.cpp
    imgui_demo.cpp
	misc/cpp/imgui_stdlib.cpp
	misc/freetype/imgui_freetype.cpp
)

target_include_directories(imgui PUBLIC
 $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
 $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/backends>
 $<INSTALL_INTERFACE:include/imgui>
 $<INSTALL_INTERFACE:include/imgui/backends>)

find_package(freetype CONFIG REQUIRED)
find_package(EASTL CONFIG REQUIRED)
target_link_libraries(imgui PRIVATE freetype EASTL)

if (BUILD_SHARED_LIBS)
    target_compile_definitions(imgui PUBLIC -DIMGUI_IMPORTS=1 PRIVATE -DIMGUI_EXPORTS=1)
endif()

include(GNUInstallDirs)

get_property(is_multi_config GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)
if (is_multi_config)
    set(INSTALL_CONFIG $<CONFIG>)
elseif (CMAKE_BUILD_TYPE)
    set(INSTALL_CONFIG ${CMAKE_BUILD_TYPE})
else ()
    set(INSTALL_CONFIG Release)
endif ()

include(CMakePackageConfigHelpers)

install(DIRECTORY backends misc DESTINATION include/imgui FILES_MATCHING PATTERN "*.h")
install(FILES imgui.h imgui_internal.h imconfig.h imstb_rectpack.h imstb_textedit.h imstb_truetype.h DESTINATION include/imgui)

file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/imgui-config.cmake.in [=[
@PACKAGE_INIT@

include("${CMAKE_CURRENT_LIST_DIR}/imgui-targets.cmake")
]=])

configure_package_config_file(
  ${CMAKE_CURRENT_BINARY_DIR}/imgui-config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/imgui-config.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/
)
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/imgui-config.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/
)

install(TARGETS imgui
    EXPORT imgui-targets
    ARCHIVE DESTINATION lib/${INSTALL_CONFIG}
    LIBRARY DESTINATION lib/${INSTALL_CONFIG}
    INCLUDES DESTINATION include)
install(EXPORT imgui-targets DESTINATION lib/cmake FILE imgui-targets.cmake)