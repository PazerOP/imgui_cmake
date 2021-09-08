cmake_minimum_required(VERSION 3.13)

project(
	imgui
	VERSION 1.80
	DESCRIPTION "CMake wrapper for Dear ImGui"
	HOMEPAGE_URL https://github.com/PazerOP/imgui_cmake
	LANGUAGES CXX
)

if (POLICY CMP0077)
	# Allow CMake 3.13+ to override options when using FetchContent / add_subdirectory.
	cmake_policy(SET CMP0077 NEW)
endif ()

option(imgui_USE_OPENGL2 "Includes the OpenGL2 implementation." off)
option(imgui_USE_OPENGL3 "Includes the OpenGL3 implementation." off)
option(imgui_USE_GLBINDING "Uses glbinding for the OpenGL implementation." off)
option(imgui_USE_SDL2 "Uses the SDL2 implementation." off)

add_library(imgui
	"imgui/imconfig.h"
	"imgui/imgui.cpp"
	"imgui/imgui.h"
	"imgui/imgui_demo.cpp"
	"imgui/imgui_draw.cpp"
	"imgui/imgui_internal.h"
	"imgui/imgui_tables.cpp"
	"imgui/imgui_widgets.cpp"
	"imgui/imstb_rectpack.h"
	"imgui/imstb_textedit.h"
	"imgui/imstb_truetype.h"
	"imgui/misc/cpp/imgui_stdlib.cpp"
	"imgui/misc/cpp/imgui_stdlib.h"
)

if (BUILD_SHARED_LIBS)
	include(GenerateExportHeader)
	generate_export_header(imgui EXPORT_MACRO_NAME IMGUI_API)
	target_precompile_headers(imgui PUBLIC "${CMAKE_CURRENT_BINARY_DIR}/imgui_export.h")
endif()

if(imgui_USE_OPENGL2)
	target_compile_definitions(imgui PUBLIC "IMGUI_USE_OPENGL2")
	target_sources(imgui PRIVATE
		"imgui/backends/imgui_impl_opengl2.cpp"
		"imgui/backends/imgui_impl_opengl2.h"
	)

	if (WIN32)
		target_link_libraries(imgui PRIVATE "OpenGL32.lib")
	endif()
endif()

if(imgui_USE_OPENGL3)
	target_compile_definitions(imgui PUBLIC "IMGUI_USE_OPENGL3")
	target_sources(imgui PRIVATE
		"imgui/backends/imgui_impl_opengl3.cpp"
		"imgui/backends/imgui_impl_opengl3.h"
	)
endif()

if(imgui_USE_GLBINDING)
	find_package(glbinding CONFIG REQUIRED)
	target_link_libraries(imgui PRIVATE glbinding::glbinding)
	target_compile_definitions(imgui PUBLIC IMGUI_USE_GLBINDING)
elseif(imgui_USE_GLAD2)
	find_package(glad2 REQUIRED)
	target_link_libraries(imgui PRIVATE glad2::glad2)
	target_compile_definitions(imgui PUBLIC IMGUI_USE_GLAD2)
endif()

if(imgui_USE_SDL2)
	find_package(SDL2 CONFIG REQUIRED)
	target_link_libraries(imgui PRIVATE SDL2::SDL2)
	target_compile_definitions(imgui PUBLIC IMGUI_USE_SDL2)
	target_sources(imgui PRIVATE
		"imgui/backends/imgui_impl_sdl.cpp"
		"imgui/backends/imgui_impl_sdl.h"
	)
elseif(imgui_USE_GLFW)
	find_package(glfw3 CONFIG REQUIRED)
	target_link_libraries(imgui PRIVATE glfw)
	target_compile_definitions(imgui PUBLIC IMGUI_USE_GLFW)
	target_sources(imgui PRIVATE
		"imgui/backends/imgui_impl_glfw.cpp"
		"imgui/backends/imgui_impl_glfw.h"
	)
endif()

target_include_directories(imgui PUBLIC
	"$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/imgui>" "$<INSTALL_INTERFACE:include/imgui>"
	"$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/imgui/misc/cpp>" "$<INSTALL_INTERFACE:include/imgui/misc/cpp>"
)

###########################################
# "install" is intended for vcpkg support #
###########################################
include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

configure_package_config_file(
	cmake/config.cmake.in
	"${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake"
	INSTALL_DESTINATION "${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}"
)

write_basic_package_version_file(
	"${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake"
	VERSION ${PROJECT_VERSION}
	COMPATIBILITY SameMajorVersion
)

install(
	FILES
		"${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake"
		"${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake"
	DESTINATION
		"${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}"
)


install(TARGETS imgui EXPORT ${PROJECT_NAME}_targets)
install(
	EXPORT ${PROJECT_NAME}_targets
	# NAMESPACE mh::
	DESTINATION "${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}"
)

file(GLOB IMGUI_HEADERS "imgui/*.h")
file(GLOB IMGUI_HEADERS_MISC "imgui/misc/cpp/*.h")
install(FILES ${IMGUI_HEADERS} ${IMGUI_HEADERS_MISC}
	DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/imgui/"
)
