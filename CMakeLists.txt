cmake_minimum_required(VERSION 2.8)
project(tangram)

# helper functions
include(${PROJECT_SOURCE_DIR}/tangram-es/toolchains/utils.cmake)

# cmake output configuration
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

if(NOT EXISTS "${PROJECT_SOURCE_DIR}/tangram-es/external/alfons/CMakeLists.txt")
    message(SEND_ERROR "Missing submodules - Please run:\n 'git submodule update --init --recursive'")
    return()
endif()

# common compiler options
set(EXECUTABLE_NAME "pi-gps")


# add sources and include headers
find_sources_and_include_directories(
    ${PROJECT_SOURCE_DIR}/src/*.h 
    ${PROJECT_SOURCE_DIR}/src/*.cpp)


add_definitions(-DPLATFORM_RPI)

# compiler options
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -fpermissive -g")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_CXX_FLAGS} -L/opt/vc/lib/ -lGLESv2 -lEGL -lbcm_host -lvchiq_arm -lvcos -lrt -lpthread")
set(CXX_FLAGS_DEBUG "-g -O0")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")

# include headers for rpi-installed libraries
include_directories(/opt/vc/include/)
include_directories(/opt/vc/include/interface/vcos/pthreads)
include_directories(/opt/vc/include/interface/vmcs_host/linux)
include_directories(SYSTEM "/usr/include/fontconfig")

# add sources and include headers
find_sources_and_include_directories(
    ${PROJECT_SOURCE_DIR}/tangram-es/linux/src/urlWorker.*
    ${PROJECT_SOURCE_DIR}/tangram-es/linux/src/urlWorker.*)

# add sources and include headers
find_sources_and_include_directories(
    ${PROJECT_SOURCE_DIR}/tangram-es/core/common/platform_gl.h
    ${PROJECT_SOURCE_DIR}/tangram-es/core/common/platform_gl.cpp)

# load core library
add_subdirectory(${PROJECT_SOURCE_DIR}/tangram-es/external)
add_subdirectory(${PROJECT_SOURCE_DIR}/tangram-es/core)
include_directories(${CORE_INCLUDE_DIRS})
include_directories(${CORE_LIBRARIES_INCLUDE_DIRS})

# add executable
add_executable(${EXECUTABLE_NAME} ${SOURCES})

# link libraries
target_link_libraries(${EXECUTABLE_NAME} ${CORE_LIBRARY})
target_link_libraries(${EXECUTABLE_NAME} -lcurl)
target_link_libraries(${EXECUTABLE_NAME} -lfontconfig)
target_link_libraries(${EXECUTABLE_NAME} -lfreetype)

add_resources(${EXECUTABLE_NAME} "${PROJECT_SOURCE_DIR}/tangram-es/scenes")

install(TARGETS ${EXECUTABLE_NAME} RUNTIME DESTINATION bin)
