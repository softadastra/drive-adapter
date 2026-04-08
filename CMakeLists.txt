cmake_minimum_required(VERSION 3.20)

if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
    project(drive_adapter
        VERSION 0.1.0
        DESCRIPTION "Drive adapter for Softadastra integration"
        LANGUAGES CXX
    )
endif()

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# ========================
# Options
# ========================
option(DRIVE_ADAPTER_BUILD_TESTS "Build drive-adapter tests" OFF)
option(DRIVE_ADAPTER_BUILD_DOCS "Build drive-adapter documentation" OFF)
option(DRIVE_WARNINGS_AS_ERRORS "Treat warnings as errors" OFF)

# ========================
# Dependencies
# ========================
find_package(softadastra REQUIRED)

# ========================
# Sources
# ========================
file(GLOB_RECURSE DRIVE_ADAPTER_SOURCES
    CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
)

# ========================
# Library
# ========================
add_library(drive_adapter STATIC ${DRIVE_ADAPTER_SOURCES})
add_library(drive::adapter ALIAS drive_adapter)

target_compile_features(drive_adapter PUBLIC cxx_std_20)

target_include_directories(drive_adapter
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

# ========================
# Softadastra links
# ========================
target_link_libraries(drive_adapter
    PUBLIC
        softadastra::core
        softadastra::wal
        softadastra::sync
        softadastra::transport
)

# ========================
# Compiler options
# ========================
if(MSVC)
    target_compile_options(drive_adapter PRIVATE /W4 /permissive-)
    if(DRIVE_WARNINGS_AS_ERRORS)
        target_compile_options(drive_adapter PRIVATE /WX)
    endif()
else()
    target_compile_options(drive_adapter PRIVATE -Wall -Wextra -Wpedantic)
    if(DRIVE_WARNINGS_AS_ERRORS)
        target_compile_options(drive_adapter PRIVATE -Werror)
    endif()
endif()

# ========================
# Target properties
# ========================
set_target_properties(drive_adapter PROPERTIES
    OUTPUT_NAME drive_adapter
    VERSION ${PROJECT_VERSION}
    SOVERSION 0
    EXPORT_NAME adapter
)

# ========================
# Install
# ========================
install(
    TARGETS drive_adapter
    EXPORT driveAdapterTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(
    DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include/"
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(
    EXPORT driveAdapterTargets
    FILE driveAdapterTargets.cmake
    NAMESPACE drive::
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/drive_adapter"
)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/drive_adapterConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/drive_adapterConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/drive_adapterConfig.cmake"
    INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/drive_adapter"
)

install(
    FILES
        "${CMAKE_CURRENT_BINARY_DIR}/drive_adapterConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/drive_adapterConfigVersion.cmake"
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/drive_adapter"
)

# ========================
# Tests
# ========================
if(DRIVE_ADAPTER_BUILD_TESTS)
    include(CTest)
    if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/tests/CMakeLists.txt")
        add_subdirectory(tests)
    endif()
endif()

# ========================
# Summary
# ========================
message(STATUS "------------------------------------------------------")
message(STATUS "drive::adapter configured (${PROJECT_VERSION})")
message(STATUS "Include dir:      ${CMAKE_CURRENT_SOURCE_DIR}/include")
message(STATUS "Build tests:      ${DRIVE_ADAPTER_BUILD_TESTS}")
message(STATUS "Build docs:       ${DRIVE_ADAPTER_BUILD_DOCS}")
message(STATUS "Warnings=error:   ${DRIVE_WARNINGS_AS_ERRORS}")
message(STATUS "Build type:       ${CMAKE_BUILD_TYPE}")
message(STATUS "Install prefix:   ${CMAKE_INSTALL_PREFIX}")
message(STATUS "------------------------------------------------------")
