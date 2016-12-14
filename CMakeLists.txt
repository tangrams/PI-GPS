cmake_minimum_required(VERSION 2.8)
project(tangram)

# helper functions
include(${PROJECT_SOURCE_DIR}/tangram-es/toolchains/utils.cmake)

# cmake output configuration
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# compiler options
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -fpermissive -g")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_CXX_FLAGS} -L/opt/vc/lib/ -lGLESv2 -lEGL -lbcm_host -lvchiq_arm -lvcos -lrt -lpthread")
set(CXX_FLAGS_DEBUG "-g -O0")
set(EXECUTABLE_NAME "pi-gps")

add_definitions(-DPLATFORM_RPI)

# check for c++11 compiler
execute_process(COMMAND ${CMAKE_CXX_COMPILER} -dumpversion OUTPUT_VARIABLE GCC_VERSION)

if("${CMAKE_CXX_COMPILER_ID}" MATCHES "GNU")
    if(NOT (GCC_VERSION VERSION_GREATER 4.8 OR GCC_VERSION VERSION_EQUAL 4.8))
        message(FATAL_ERROR "Please install g++ version 4.8 or greater")
    else()
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    endif()
else()
    message(FATAL_ERROR "Please install a C++11 compatible compiler")
endif()

# add sources and include headers
find_sources_and_include_directories(
    ${PROJECT_SOURCE_DIR}/src/*.h 
    ${PROJECT_SOURCE_DIR}/src/*.cpp)

# add sources and include headers
find_sources_and_include_directories(
    ${PROJECT_SOURCE_DIR}/tangram-es/linux/src/urlWorker.*
    ${PROJECT_SOURCE_DIR}/tangram-es/linux/src/urlWorker.*)

# add sources and include headers
find_sources_and_include_directories(
    ${PROJECT_SOURCE_DIR}/tangram-es/common/platform_gl.h
    ${PROJECT_SOURCE_DIR}/tangram-es/core/common/platform_gl.cpp)

# include headers for rpi-installed libraries
include_directories(/opt/vc/include/)
include_directories(/opt/vc/include/interface/vcos/pthreads)
include_directories(/opt/vc/include/interface/vmcs_host/linux)

# load core library
add_subdirectory(${PROJECT_SOURCE_DIR}/tangram-es/core)
include_directories(${CORE_INCLUDE_DIRS})
include_directories(${CORE_LIBRARIES_INCLUDE_DIRS})

# add executable
add_executable(${EXECUTABLE_NAME} ${SOURCES})

# copy resources
file(GLOB_RECURSE RESOURCES ${PROJECT_SOURCE_DIR}/tangram-es/core/resources/*)
foreach(_resource ${RESOURCES})
    file(COPY ${_resource} DESTINATION ${PROJECT_SOURCE_DIR}/build/bin)
endforeach()

# link libraries
target_link_libraries(${EXECUTABLE_NAME} -lcurl)
target_link_libraries(${EXECUTABLE_NAME} core)

add_resources(${EXECUTABLE_NAME} "${PROJECT_SOURCE_DIR}/tangram-es/scenes")
