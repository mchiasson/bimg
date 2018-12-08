cmake_minimum_required(VERSION 3.2)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

include(Hunter/HunterGate)
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.23.30.tar.gz"
    SHA1 "c062b0a2316cf8dcb57b6a9ad49fea1025a928f6"
    LOCAL
)

project(bimg VERSION 1.0.0)

file(GLOB BIMG_SOURCES src/*)
file(GLOB ASTC_SOURCES 3rdparty/astc/*)
file(GLOB EDTAA3_SOURCES 3rdparty/edtaa3/*)
file(GLOB LIBSQUISH_SOURCES 3rdparty/libsquish/*)
file(GLOB IQA_SOURCES 3rdparty/iqa/source/*)
file(GLOB_RECURSE NVTT_SOURCES 3rdparty/nvtt/*)
file(GLOB PVRTC_SOURCES 3rdparty/pvrtc/*)

add_library(bimg
    ${BIMG_SOURCES}
    ${ASTC_SOURCES}
    ${EDTAA3_SOURCES}
    ${LIBSQUISH_SOURCES}
    ${IQA_SOURCES}
    ${NVTT_SOURCES}
    ${PVRTC_SOURCES}
)

target_include_directories(bimg
    PUBLIC
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    PRIVATE
        3rdparty
        3rdparty/nvtt
        3rdparty/iqa/include
)

hunter_add_package(bx)
find_package(bx CONFIG REQUIRED)
target_link_libraries(bimg PUBLIC bx::bx)

set(targets_export_name "bimgTargets")
set(ConfigPackageLocation lib/cmake/bimg)

# Create the CMake version file.
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/bimgConfigVersion.cmake
    VERSION ${PACKAGE_VERSION}
    COMPATIBILITY AnyNewerVersion
)

# Create the Config file.
include(CMakePackageConfigHelpers)
configure_package_config_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake/bimgConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/bimgConfig.cmake
    INSTALL_DESTINATION ${ConfigPackageLocation}
)

# Install the different headers and libraries.
install(
    DIRECTORY include/bimg
    DESTINATION include COMPONENT Development
)

install(TARGETS bimg
    EXPORT ${targets_export_name}
    LIBRARY DESTINATION lib COMPONENT Runtime
    ARCHIVE DESTINATION lib COMPONENT Development
    RUNTIME DESTINATION bin COMPONENT Runtime
    BUNDLE DESTINATION bin COMPONENT Runtime
    PUBLIC_HEADER DESTINATION include COMPONENT Development
    BUNDLE DESTINATION bin COMPONENT Runtime
)

# Install the generated CMake files.
install(
    EXPORT ${targets_export_name}
    NAMESPACE "bimg::"
    DESTINATION ${ConfigPackageLocation}
)

install(
    FILES
        ${CMAKE_CURRENT_BINARY_DIR}/bimgConfigVersion.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/bimgConfig.cmake
    DESTINATION
        ${ConfigPackageLocation}
)
