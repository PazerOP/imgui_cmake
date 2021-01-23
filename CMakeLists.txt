cmake_minimum_required(VERSION 3.13)

project(imgui)

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

if(imgui_USE_OPENGL2)
	target_sources(imgui PRIVATE "imgui/backends/imgui_impl_opengl2.cpp")
	target_sources(imgui PRIVATE "imgui/backends/imgui_impl_opengl2.h")

	target_compile_definitions(imgui PUBLIC "IMGUI_USE_OPENGL2")

	if (WIN32)
		target_link_libraries(imgui PUBLIC "OpenGL32.lib")
	endif()
endif()

if(imgui_USE_OPENGL3)
	target_sources(imgui PRIVATE "imgui/backends/imgui_impl_opengl3.cpp")
	target_sources(imgui PRIVATE "imgui/backends/imgui_impl_opengl3.h")

	target_compile_definitions(imgui PUBLIC "IMGUI_USE_OPENGL3")
endif()

if(imgui_USE_GLBINDING)
	find_package(glbinding CONFIG REQUIRED)
	target_link_libraries(imgui PUBLIC glbinding::glbinding glbinding::glbinding-aux)
	target_compile_definitions(imgui PUBLIC IMGUI_USE_GLBINDING)
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
	"imgui"
	"imgui/misc/cpp"
)
