cmake_minimum_required(VERSION 3.21)

project(
    ccld_GameSettingsOverride
    VERSION 2.2.0
    DESCRIPTION "SKSE plugin that allows users to modify game settings via configuration files."
    LANGUAGES CXX
)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_INTERPROCEDURAL_OPTIMIZATION "$<$<CONFIG:Release>:ON>")

# Disable CommonLibSSE-NG tests.
set(BUILD_TESTS OFF CACHE BOOL "" FORCE)

# Use the working HelloWorld template's local CommonLibSSE-NG.
add_subdirectory(lib/commonlibsse-ng)
include(lib/commonlibsse-ng/cmake/CommonLibSSE.cmake)
include(cmake/headerlist.cmake)
include(cmake/sourcelist.cmake)

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/version.rc.in"
    "${CMAKE_CURRENT_BINARY_DIR}/version.rc"
    @ONLY
)

# -- Declare Sources -----------------------------------------------------------

source_group(
    TREE
        "${CMAKE_CURRENT_SOURCE_DIR}"
    FILES
        ${PROJECT_HEADERS}
        ${PROJECT_SOURCES}
)

# -- Declare Dependencies ------------------------------------------------------
find_package(fmt CONFIG REQUIRED)
find_package(spdlog CONFIG REQUIRED)
find_package(tomlplusplus CONFIG REQUIRED)

# Build the original plugin.

add_commonlibsse_plugin(
    "${PROJECT_NAME}"
    SOURCES
        ${PROJECT_HEADERS}
        ${PROJECT_SOURCES}
        "${CMAKE_CURRENT_BINARY_DIR}/version.rc"
)

target_compile_definitions(
    "${PROJECT_NAME}"
    PRIVATE
        _UNICODE
        UNICODE
        NOMINMAX
)

target_compile_features(
    "${PROJECT_NAME}"
    PRIVATE
        cxx_std_23
)

if(MSVC)
    target_compile_options(
        "${PROJECT_NAME}"
        PRIVATE
            /EHsc
            /permissive-
            /sdl
            /utf-8
            /W4
            /Zc:__cplusplus
            /Zc:preprocessor
    )
endif()

target_include_directories(
    "${PROJECT_NAME}"
    PRIVATE
        "${CMAKE_CURRENT_SOURCE_DIR}/src"
)

target_link_libraries(
    "${PROJECT_NAME}"
    PRIVATE
        fmt::fmt
        spdlog::spdlog
        tomlplusplus::tomlplusplus
)

target_precompile_headers(
    "${PROJECT_NAME}"
    PRIVATE
        "src/XSEPlugin/PCH.h"
)

# Automatically deploy the built DLL, like the HelloWorld template.
if(DEFINED OUTPUT_FOLDER)
    set(DLL_FOLDER "${OUTPUT_FOLDER}/SKSE/Plugins")

    message(STATUS "SKSE plugin output folder: ${DLL_FOLDER}")

    add_custom_command(
        TARGET "${PROJECT_NAME}"
        POST_BUILD
        COMMAND "${CMAKE_COMMAND}" -E make_directory
            "${DLL_FOLDER}"
        COMMAND "${CMAKE_COMMAND}" -E copy_if_different
            "$<TARGET_FILE:${PROJECT_NAME}>"
            "${DLL_FOLDER}/$<TARGET_FILE_NAME:${PROJECT_NAME}>"
        VERBATIM
    )

    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        add_custom_command(
            TARGET "${PROJECT_NAME}"
            POST_BUILD
            COMMAND "${CMAKE_COMMAND}" -E copy_if_different
                "$<TARGET_PDB_FILE:${PROJECT_NAME}>"
                "${DLL_FOLDER}/$<TARGET_PDB_FILE_NAME:${PROJECT_NAME}>"
            VERBATIM
        )
    endif()
endif()
