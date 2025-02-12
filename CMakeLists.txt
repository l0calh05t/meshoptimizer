cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
project(meshoptimizer VERSION 0.12 LANGUAGES CXX)

option(BUILD_DEMO "Build demo" OFF)
option(BUILD_TOOLS "Build tools" OFF)
option(BUILD_SHARED_LIBS "Build shared libraries" OFF)

set(SOURCES
    src/meshoptimizer.h
    src/allocator.cpp
    src/clusterizer.cpp
    src/indexcodec.cpp
    src/indexgenerator.cpp
    src/overdrawanalyzer.cpp
    src/overdrawoptimizer.cpp
    src/simplifier.cpp
    src/spatialorder.cpp
    src/stripifier.cpp
    src/vcacheanalyzer.cpp
    src/vcacheoptimizer.cpp
    src/vertexcodec.cpp
    src/vfetchanalyzer.cpp
    src/vfetchoptimizer.cpp
)

add_library(meshoptimizer ${SOURCES})
target_include_directories(meshoptimizer INTERFACE "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>")

if(MSVC)
    target_compile_options(meshoptimizer PRIVATE /W4 /WX)
else()
    target_compile_options(meshoptimizer PRIVATE -Wall -Wextra -Werror)
endif()

if(BUILD_SHARED_LIBS)
    set_target_properties(
        meshoptimizer
        PROPERTIES
        CXX_VISIBILITY_PRESET hidden
        VISIBILITY_INLINES_HIDDEN ON
    )
    if(WIN32)
        target_compile_definitions(
            meshoptimizer
            INTERFACE
            "MESHOPTIMIZER_API=__declspec(dllimport)"
            PRIVATE
            "MESHOPTIMIZER_API=__declspec(dllexport)"
        )
    else()
        target_compile_definitions(
            meshoptimizer
            PUBLIC
            "MESHOPTIMIZER_API=__attribute__((visibility(\"default\")))"
        )
    endif()
endif()

set(TARGETS meshoptimizer)

if(BUILD_DEMO)
    add_executable(demo demo/main.cpp demo/miniz.cpp demo/tests.cpp tools/meshloader.cpp)
    target_link_libraries(demo meshoptimizer)
endif()

if(BUILD_TOOLS)
    add_executable(gltfpack tools/gltfpack.cpp tools/meshloader.cpp)
    target_link_libraries(gltfpack meshoptimizer)
    list(APPEND TARGETS gltfpack)

    if(MSVC AND NOT (CMAKE_VERSION VERSION_LESS "3.1"))
        install(
            FILES $<TARGET_PDB_FILE:gltfpack>
            DESTINATION bin
            OPTIONAL
        )
    endif()

    set_target_properties(gltfpack PROPERTIES INSTALL_RPATH "$ORIGIN/../lib")
endif()

install(
    TARGETS ${TARGETS}
    EXPORT meshoptimizerTargets
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    INCLUDES DESTINATION include
)

install(
    FILES src/meshoptimizer.h
    DESTINATION include
)

if(MSVC AND BUILD_SHARED_LIBS AND NOT (CMAKE_VERSION VERSION_LESS "3.1"))
    install(
        FILES $<TARGET_PDB_FILE:meshoptimizer>
        DESTINATION bin
        OPTIONAL
    )
endif()

install(
    EXPORT meshoptimizerTargets
    DESTINATION lib/cmake/meshoptimizer
    NAMESPACE meshoptimizer::
)

include(CMakePackageConfigHelpers)

configure_package_config_file(
    meshoptimizerConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/meshoptimizerConfig.cmake
    INSTALL_DESTINATION lib/cmake/meshoptimizer
    NO_SET_AND_CHECK_MACRO
)

write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/meshoptimizerConfigVersion.cmake
    COMPATIBILITY ExactVersion
)

install(
    FILES
    ${CMAKE_CURRENT_BINARY_DIR}/meshoptimizerConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/meshoptimizerConfigVersion.cmake
    DESTINATION lib/cmake/meshoptimizer
)
