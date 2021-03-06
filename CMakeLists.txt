cmake_minimum_required(VERSION 2.8 FATAL_ERROR)
cmake_policy(VERSION 2.8)

project(ogl-samples-pack)
enable_testing()

#CMAKE_BUILD_TOOL

if(CMAKE_CURRENT_SOURCE_DIR EQUAL CMAKE_CURRENT_BINARY_DIR)
	message(FATAL_ERROR "Source and binary directories must be different")
endif(CMAKE_CURRENT_SOURCE_DIR EQUAL CMAKE_CURRENT_BINARY_DIR)

################################
# Add GLEXT 

find_package(OpenGL)
if(WIN32)
	set(GL_LIBRARY OpenGL32.lib)
elseif(APPLE)
	set(GL_LIBRARY ${OPENGL_LIBRARY})
elseif(UNIX)
	set(GL_LIBRARY GL)
endif(WIN32)

if(NOT OPENGL_FOUND)
	message("ERROR: OpenGL not found")
endif(NOT OPENGL_FOUND)

function(getKhronosGLHeader HEADER)
	file(DOWNLOAD http://www.opengl.org/registry/api/GL/${HEADER}
		khronos-headers/GL/${HEADER})
endfunction(getKhronosGLHeader)

#download Updated OpenGL Extension headers.
getKhronosGLHeader(glext.h)
getKhronosGLHeader(glcorearb.h)
getKhronosGLHeader(glxext.h)
getKhronosGLHeader(wglext.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR}/khronos-headers/)

################################
# Add GLEW

set(GLEW_DIRECTORY glew-1.10.0)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/external/${GLEW_DIRECTORY}/include)

if(APPLE)
	find_library(GLEW_LIBRARY GLEW)
else()
	add_definitions(-DGLEW_STATIC)
	include_directories(${CMAKE_CURRENT_SOURCE_DIR}/external/${GLEW_DIRECTORY}/include)
	add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/external/${GLEW_DIRECTORY})
	set(GLEW_LIBRARY ${GLEW_LIBRARIES})
	link_directories(${CMAKE_CURRENT_BINARY_DIR}/GLEW-prefix/)
endif()

################################
# Add GLFW

set(GLFW_DIRECTORY glfw-3.0.3)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/external/${GLFW_DIRECTORY}/include)

if(APPLE)
	find_library(GLFW_LIBRARY GLFW)
else()
	set(GLFW_INSTALL OFF)
	set(GLFW_BUILD_EXAMPLES OFF)
	set(GLFW_BUILD_TESTS OFF)
	add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/external/${GLFW_DIRECTORY})
	include_directories(${CMAKE_CURRENT_SOURCE_DIR}/external/${GLFW_DIRECTORY}/include)
	set(GLFW_LIBRARY ${GLFW_LIBRARIES})
	list(APPEND GLFW_LIBRARY glfw3 ${GLFW_LIBRARIES} )
endif()

################################
# Add Cocoa
if(APPLE)
	find_library(COCOA_LIBRARY cocoa)
	find_library(IOKIT_LIBRARY iokit)
endif()

################################
# Add GLM 

set(GLM_DIRECTORY glm-0.9.5.A)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/external/${GLM_DIRECTORY})

################################
# Add GLI 

set(GLI_DIRECTORY gli-0.4.1.0)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/external/${GLI_DIRECTORY})

################################
# Add GLO 

set(GLO_DIRECTORY glo-0.1.0.0)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/external/${GLO_DIRECTORY})

################################
# Add GLF

set(GLF_DIRECTORY glf)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/external/)

################################
# Add OpenGL ES

set(ES_DIRECTORY gles-2.0)

if(MSVC AND NOT CMAKE_CL_64)
	set(ES_BINARY_DIRECTORY "win32-vc")
endif(MSVC AND NOT CMAKE_CL_64)

if(MSVC AND CMAKE_CL_64)
	set(ES_BINARY_DIRECTORY "win64-vc")
endif(MSVC AND CMAKE_CL_64)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/external/${ES_DIRECTORY})
if(WIN32)
	link_directories(${CMAKE_CURRENT_SOURCE_DIR}/external/${ES_DIRECTORY}/lib/${ES_BINARY_DIRECTORY})
endif()

set(ES_LIBRARY libGLESv2)

################################
# Add EGL

set(EGL_DIRECTORY egl-1.4)

if(MSVC AND NOT CMAKE_CL_64)
	set(EGL_BINARY_DIRECTORY "win32-vc")
endif(MSVC AND NOT CMAKE_CL_64)

if(MSVC AND CMAKE_CL_64)
	set(EGL_BINARY_DIRECTORY "win64-vc")
endif(MSVC AND CMAKE_CL_64)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/external/${EGL_DIRECTORY})
if(WIN32)
	link_directories(${CMAKE_CURRENT_SOURCE_DIR}/external/${EGL_DIRECTORY}/lib/${EGL_BINARY_DIRECTORY})
endif()

set(EGL_LIBRARY libEGL)

################################
# Add libraries to executables

set(BINARY_FILES 
	${GL_LIBRARY} 
	GLEW
	${GLEW_LIBRARIES}
	glfw
	${GLFW_LIBRARIES}
	${GLM_LIBRARY} 
	${GLI_LIBRARY}
	${COCOA_LIBRARY}
	${IOKIT_LIBRARY})

################################
# Add output directory

if(MSVC)
	set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR}/build/)
else(MSVC)
	if(CMAKE_BUILD_TYPE STREQUAL "Debug")
		set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR}/build/debug )
	else(CMAKE_BUILD_TYPE STREQUAL "Debug")
		set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR}/build/release )
	endif(CMAKE_BUILD_TYPE STREQUAL "Debug")
endif(MSVC)

################################
# Add custom target to copy all data

if(APPLE)
	add_custom_target(
		${TARGET_DATA_COPY}
 		COMMAND ls)

else(APPLE)
	set(TARGET_DATA_COPY DATA_COPY)
	if(${CMAKE_CURRENT_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_BINARY_DIR})
		add_custom_target(
			${TARGET_DATA_COPY}
 			COMMAND ${CMAKE_COMMAND} -E echo "In source build")
	else(${CMAKE_CURRENT_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_BINARY_DIR})
		add_custom_target(
			${TARGET_DATA_COPY}
 			COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR}/data ${CMAKE_CURRENT_BINARY_DIR}/data)
	endif(${CMAKE_CURRENT_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_BINARY_DIR})
endif(APPLE)

set(OGL_DATA_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/data)

################################
# Add options

option(OGL_SAMPLES_STATUS "OGL_SAMPLES_STATUS" OFF)
if(OGL_SAMPLES_STATUS)
	add_definitions(-DGLF_AUTO_STATUS)
endif()


if(CMAKE_COMPILER_IS_GNUCC OR CMAKE_C_COMPILER_ID STREQUAL "Clang")
	add_definitions(-fpermissive)
endif()
################################
# Add definitions

if(MSVC)
	add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif(MSVC)

################################
# function to create a project for the sample in the solution

function(createSample NAME)
	file(GLOB PROJECT_SOURCE *.cpp)
	file(GLOB_RECURSE PROJECT_SHADER *.vert *.frag *.geom *.cont *.eval *.glsl *.vs *.fs *.gs *.ps *.es)
	file(GLOB PROJECT_INLINE *.inl)
	file(GLOB PROJECT_HEADER *.hpp)
	file(GLOB PROJECT_TEXT readme.txt)

	source_group("Text Files" FILES ${PROJECT_TEXT})
	source_group("Inline Files" FILES ${PROJECT_INLINE})
	source_group("Shader Files" FILES ${PROJECT_SHADER})

	add_executable(${NAME} ${PROJECT_SOURCE} ${PROJECT_INLINE} ${PROJECT_HEADER} ${PROJECT_SHADER} ${PROJECT_TEXT})
	add_custom_command(
		TARGET ${NAME} POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR}/data ${CMAKE_CURRENT_BINARY_DIR}/data)
	add_custom_command(
		TARGET ${NAME} POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy ${ASSIMP_BINARY} ${CMAKE_CURRENT_BINARY_DIR})

	add_dependencies(${NAME} glfw GLEW)
	target_link_libraries(${NAME} ${BINARY_FILES})
	install ( TARGETS ${NAME} DESTINATION .)
endfunction(createSample)

function(createSampleWGL NAME)
	file(GLOB PROJECT_SOURCE *.cpp)
	file(GLOB_RECURSE PROJECT_SHADER *.vert *.frag *.geom *.cont *.eval *.glsl)
	file(GLOB PROJECT_INLINE *.inl)
	file(GLOB PROJECT_HEADER *.hpp)
	file(GLOB PROJECT_TEXT readme.txt)

	source_group("Text Files" FILES ${PROJECT_TEXT})
	source_group("Inline Files" FILES ${PROJECT_INLINE})
	source_group("Shader Files" FILES ${PROJECT_SHADER})

	add_executable(${NAME} WIN32 ${PROJECT_SOURCE} ${PROJECT_INLINE} ${PROJECT_HEADER} ${PROJECT_SHADER} ${PROJECT_TEXT})
	add_custom_command(
		TARGET ${NAME} POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR}/data ${CMAKE_CURRENT_BINARY_DIR}/data)
	add_custom_command(
		TARGET ${NAME} POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy ${ASSIMP_BINARY} ${CMAKE_CURRENT_BINARY_DIR})

	add_dependencies(${NAME} GLEW glfw)
	target_link_libraries(${NAME} ${BINARY_FILES})
	install ( TARGETS ${NAME} DESTINATION .)
endfunction(createSampleWGL)

################################
# Add subdirectory

add_subdirectory(samples)

install (
	DIRECTORY data
	DESTINATION .
)
# See http://www.vtk.org/Wiki/CMake:CPackPackageGenerators
set (CPACK_PACKAGE_VERSION_MAJOR "4")
set (CPACK_PACKAGE_VERSION_MINOR "4")
set (CPACK_PACKAGE_VERSION_PATCH "1")
# Use current date in YYYYMMDD format as patch number

# cpack mistakenly detects Mingw-w64 as win32
if (MINGW)
	if (CMAKE_SIZEOF_VOID_P EQUAL 8)
		set (CPACK_SYSTEM_NAME win64)
	endif ()
endif ()


if (WIN32)
	set (CPACK_GENERATOR "ZIP")
else ()
	set (CPACK_GENERATOR "TBZ2")
endif ()

include(CPack)
