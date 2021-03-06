set(SDL_VERSION "2" CACHE STRING "Version of SDL to use (1.2 or 2)")
set(BUILD_GL ON CACHE STRING "Build with OpenGL")

if (SDL_VERSION EQUAL "2")
	include(FindPkgConfig)
	pkg_search_module(SDL2 sdl2)
	if (SDL2_FOUND)
		set(SDL_INCLUDE_DIR ${SDL2_INCLUDE_DIRS} CACHE INTERNAL "")
		set(SDL_LIBRARY ${SDL2_LIBRARIES} CACHE INTERNAL "")
		set(SDLMAIN_LIBRARY "" CACHE INTERNAL "")
		link_directories(${SDL2_LIBDIR})
		set(SDL_VERSION_DEBIAN "2-2.0-0")
	endif()
endif()

if(SDL_VERSION EQUAL "1.2" OR NOT SDL2_FOUND)
	find_package(SDL 1.2)
	set(SDL_VERSION "1.2" PARENT_SCOPE)
	set(SDL_VERSION_DEBIAN "1.2debian")
endif()

if (NOT SDL2_FOUND AND NOT SDL_FOUND)
	set(BUILD_SDL OFF PARENT_SCOPE)
	return()
endif()

set(CPACK_DEBIAN_PACKAGE_DEPENDS "${CPACK_DEBIAN_PACKAGE_DEPENDS},libsdl${SDL_VERSION_DEBIAN}" PARENT_SCOPE)

file(GLOB PLATFORM_SRC ${CMAKE_SOURCE_DIR}/src/platform/sdl/sdl-*.c)
set(PLATFORM_LIBRARY ${SDL_LIBRARY} ${SDLMAIN_LIBRARY})
include_directories(${CMAKE_SOURCE_DIR}/src/platform/sdl ${SDL_INCLUDE_DIR})

set(MAIN_SRC ${CMAKE_SOURCE_DIR}/src/platform/sdl/main.c)

if(BUILD_RASPI)
	add_definitions(-DBUILD_RASPI)
	set(EGL_MAIN_SRC ${CMAKE_SOURCE_DIR}/src/platform/sdl/egl-sdl.c)
	set(EGL_LIBRARY "-lEGL -lGLESv2 -lbcm_host")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fgnu89-inline")
	add_executable(${BINARY_NAME}-rpi ${PLATFORM_SRC} ${MAIN_SRC} ${EGL_MAIN_SRC})
	target_link_libraries(${BINARY_NAME}-rpi ${BINARY_NAME} ${PLATFORM_LIBRARY} ${EGL_LIBRARY})
	install(TARGETS ${BINARY_NAME}-rpi DESTINATION bin COMPONENT ${BINARY_NAME}-rpi)
endif()

if(BUILD_BBB OR BUILD_RASPI OR NOT BUILD_GL)
	list(APPEND MAIN_SRC ${CMAKE_SOURCE_DIR}/src/platform/sdl/sw-sdl.c)
else()
	list(APPEND MAIN_SRC ${CMAKE_SOURCE_DIR}/src/platform/sdl/gl-sdl.c)
	add_definitions(-DBUILD_GL)
	find_package(OpenGL REQUIRED)
	include_directories(${OPENGL_INCLUDE_DIR})
endif()

add_executable(${BINARY_NAME}-sdl WIN32 ${PLATFORM_SRC} ${MAIN_SRC})
target_link_libraries(${BINARY_NAME}-sdl ${BINARY_NAME} ${PLATFORM_LIBRARY} ${OPENGL_LIBRARY})
set_target_properties(${BINARY_NAME}-sdl PROPERTIES OUTPUT_NAME ${BINARY_NAME})
install(TARGETS ${BINARY_NAME}-sdl DESTINATION bin COMPONENT ${BINARY_NAME}-sdl)
