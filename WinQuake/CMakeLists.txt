cmake_minimum_required(VERSION 2.8)
project(QUAKE C)


# compilers
if(MSVC)
    set(CMAKE_C_FLAGS "/WX /W4 /EHsc /wd4201 /wd4996")
    set(CMAKE_CXX_FLAGS "/WX /W4 /EHsc /wd4201")
else()
    # using Clang, GNU or Intel
    set(CMAKE_C_FLAGS "-Wall -Wextra -Werror -fPIC")
    set(CMAKE_CXX_FLAGS "-Wall -Wextra -Werror -fPIC")

	# clang static analyzer
	if(STATIC_ANALYZER)
	    set(CMAKE_C_FLAGS "--analyze -S -Qunused-arguments")
	    set(CMAKE_CXX_FLAGS "--analyze -S -Qunused-arguments")
	    set(CMAKE_C_FLAGS_RELEASE "")
	    set(CMAKE_CXX_FLAGS_RELEASE "")
	endif()

endif(MSVC)


file(GLOB SOURCE_FILES ${QUAKE_SOURCE_DIR}/*.c)
set_source_files_properties(${SOURCE_FILES} PROPERTIES LANGUAGE C)

set(QUAKE_PLATFORM_TO_USE "WinQuake" CACHE STRING "Quake platform to use")
set(QUAKE_RENDERER_TO_USE "OpenGL" CACHE STRING "Quake renderer to use")

set(CACHE QUAKE_PLATFORM_TO_USE PROPERTY STRINGS WinQuake SDLQuake Null)
set(CACHE QUAKE_RENDERER_TO_USE PROPERTY STRINGS OpenGL Software Vulkan)

if (MSVC)
	set(SOURCE_FILES ${SOURCE_FILES} ${QUAKE_SOURCE_DIR}/winquake.rc)
endif(MSVC)

if (${QUAKE_PLATFORM_TO_USE} MATCHES "WinQuake")
	include_directories(${QUAKE_SOURCE_DIR}/platforms/win)
	file(GLOB WINQUAKE_SOURCES ${QUAKE_SOURCE_DIR}/platforms/win/*.c)
	set_source_files_properties(${WINQUAKE_SOURCES} PROPERTIES LANGUAGE C)
	set(QUAKE_EXTRA_LIBS kernel32 user32 gdi32 winspool comdlg32 advapi32 shell32 ole32 oleaut32 uuid odbc32 odbccp32 dxguid wsock32 winmm comctl32)

	set(SOURCE_FILES ${SOURCE_FILES} ${WINQUAKE_SOURCES})

elseif(${QUAKE_PLATFORM_TO_USE} MATCHES "SDLQuake")
	include_directories(${QUAKE_SOURCE_DIR}/platforms/sdl)
	include_directories("../SDL2/include")
	file(GLOB SDLQUAKE_SOURCES ${QUAKE_SOURCE_DIR}/platforms/sdl/*.c)
	set_source_files_properties(${SDLQUAKE_SOURCES} PROPERTIES LANGUAGE C)

	set(SOURCE_FILES ${SOURCE_FILES} ${SDLQUAKE_SOURCES})

elseif(${QUAKE_PLATFORM_TO_USE} MATCHES "Null")
	include_directories(${QUAKE_SOURCE_DIR}/platforms/null)

	file(GLOB NULLQUAKE_SOURCES ${QUAKE_SOURCE_DIR}/platforms/null/*.c)
	set_source_files_properties(${NULLQUAKE_SOURCES} PROPERTIES LANGUAGE C)

	set (SOURCE_FILES ${SOURCE_FILES} ${NULLQUAKE_SOURCES})
endif()

if (${QUAKE_RENDERER_TO_USE} MATCHES "OpenGL")
	add_definitions(-DGLQUAKE)
	include_directories(${QUAKE_SOURCE_DIR}/render/gl)

	file(GLOB GLQUAKE_SOURCES ${QUAKE_SOURCE_DIR}/render/gl/*.c)
	set_source_files_properties(${GLQUAKE_SOURCES} PROPERTIES LANGUAGE C)

	set(SOURCE_FILES ${SOURCE_FILES} ${GLQUAKE_SOURCES})
	set(QUAKE_EXTRA_LIBS ${QUAKE_EXTRA_LIBS} opengl32 glu32)

elseif(${QUAKE_RENDERER_TO_USE} MATCHES "Software")

	include_directories(${QUAKE_SOURCE_DIR}/render/sw)
	file(GLOB SOFTWARE_SOURCES ${QUAKE_SOURCE_DIR}/render/sw/*.c)
	set_source_files_properties(${SOFTWARE_SOURCES} PROPERTIES LANGUAGE C)

	set(SOURCE_FILES ${SOURCE_FILES} ${SOFTWARE_SOURCES})

elseif(${QUAKE_RENDERER_TO_USE} MATCHES "Vulkan")

endif()

include_directories(".")

add_executable(vkQuake WIN32 ${SOURCE_FILES})
set_target_properties(vkQuake PROPERTIES 
	LINKER_LANGUAGE C
	OUTPUT_NAME "vkQuake")

target_link_libraries(vkQuake ${QUAKE_EXTRA_LIBS})