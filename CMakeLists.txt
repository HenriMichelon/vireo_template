#
# Copyright (c) 2025-present Henri Michelon
#
# This software is released under the MIT License.
# https://opensource.org/licenses/MIT
#
cmake_minimum_required(VERSION 3.29)

#######################################################
project(vireo_template)
if (NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/.env.cmake")
    message(FATAL_ERROR "Please create a .env.cmake file with the VIREO_PROJECT_DIR variable")
endif ()
include(.env.cmake)
if (NOT DEFINED VIREO_PROJECT_DIR)
    message(FATAL_ERROR "Please set VIREO_PROJECT_DIR in the .env.cmake file")
endif()
set(DIRECTX_BACKEND ON)

#######################################################
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_COMPILE_WARNING_AS_ERROR ON)
set(CMAKE_CXX_EXTENSIONS OFF)
if(MSVC)
    set(USE_STATIC_MSVC_RUNTIME_LIBRARY OFF)
endif()
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()
if(WIN32)
    add_compile_definitions(WIN32_LEAN_AND_MEAN UNICODE _UNICODE)
endif ()

#######################################################
include(FetchContent)
include(cmake/shaders.cmake)
include(cmake/libraries.cmake)

#######################################################
add_subdirectory(${VIREO_PROJECT_DIR} external_lib_build)
set(VIREO_INCLUDE_DIR ${VIREO_PROJECT_DIR}/include)
set(VIREO_TARGET "vireo_rhi")

set(SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)
set(SHADERS_SRC_DIR ${SRC_DIR}/shaders)
set(SHADERS_INCLUDE_DIR ${SRC_DIR}/shaders/include)
set(SHADERS_BUILD_DIR ${CMAKE_CURRENT_SOURCE_DIR}/shaders)

#######################################################
# Slang shaders
file(MAKE_DIRECTORY ${SHADERS_BUILD_DIR})
file(GLOB_RECURSE SHADERS_SOURCE_FILES
        "${SHADERS_SRC_DIR}/*.slang"
)
add_shaders(shaders ${SHADERS_BUILD_DIR} ${SHADERS_INCLUDE_DIR} ${SHADERS_SOURCE_FILES} )
#add_dependencies(target_shaders ${VIREO_TARGET})

#######################################################
# Common files
set(COMMON_SOURCES
        ${SRC_DIR}/os/windows/Win32Application.cpp
)
set(COMMON_MODULES
        ${SRC_DIR}/Application.ixx
        ${SRC_DIR}/os/windows/Win32Application.ixx
)

#######################################################
function(build_target TARGET_NAME SRCS MODULES)
    message("Building target ${TARGET_NAME}")
    add_executable(${TARGET_NAME} ${COMMON_SOURCES} ${SRCS})
    target_sources(${TARGET_NAME}
        PUBLIC
        FILE_SET CXX_MODULES
        FILES
            ${COMMON_MODULES}
            ${MODULES}
    )
    compile_options(${TARGET_NAME})
    target_include_directories(${TARGET_NAME} PUBLIC ${INCLUDE_DIR} ${VIREO_INCLUDE_DIR})
    target_link_libraries(${TARGET_NAME} ${VIREO_TARGET} glm::glm glm-modules std-cxx-modules)
    add_dependencies(${TARGET_NAME} ${VIREO_TARGET})
    if(WIN32)
        target_precompile_headers(${TARGET_NAME} PRIVATE
                "<windows.h>"
                "<cstdint>"
        )
        if (DIRECTX_BACKEND)
            target_link_libraries(${TARGET_NAME} Xinput dinput8 dxguid dxgi d3d12 d3dcompiler)
        endif ()
        if(MINGW)
            target_link_options(${TARGET_NAME} PRIVATE "-mwindows")
        endif()
        set_target_properties(${TARGET_NAME} PROPERTIES
                WIN32_EXECUTABLE TRUE)
    endif()
endfunction()

#######################################################
#set(MY_TARGET_SRC
#        ${SRC_DIR}/MyApp.cpp
#        ${SRC_DIR}/MyAppMain.cpp)
#set(MY_TARGET_MODULES ${SRC_DIR}/MyApp.ixx)
#build_target(myapp "${MY_TARGET_SRC}" ${MY_TARGET_MODULES})
