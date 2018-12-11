cmake_minimum_required(VERSION 3.2)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

include(Hunter/HunterGate)
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.23.69.tar.gz"
    SHA1 "3d6b1267844f7bb64ba9981ac5a1ff78a0237d41"
    LOCAL
)

project(bimg VERSION 1.0.0)

option(BIMG_BUILD_TOOLS "Build bimg tools." ON)

################################################################################
# bimg 3rdparty libs
################################################################################

hunter_add_package(bx)
find_package(bx CONFIG REQUIRED)

hunter_add_package(stb)
find_package(stb CONFIG REQUIRED)


################################################################################
# bimg lib
################################################################################

add_library(bimg_decode src/image_decode.cpp)
target_link_libraries(bimg_decode PUBLIC bkaradzic::bx stb::stb)
target_include_directories(bimg_decode
    PUBLIC
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    PRIVATE
        3rdparty
        3rdparty/nvtt
        3rdparty/iqa/include
)

add_library(bimg_encode 
    3rdparty/astc/astc_averages_and_directions.cpp
    3rdparty/astc/astc_block_sizes2.cpp
    3rdparty/astc/astc_color_quantize.cpp
    3rdparty/astc/astc_color_unquantize.cpp
    3rdparty/astc/astc_compress_symbolic.cpp
    3rdparty/astc/astc_compute_variance.cpp
    3rdparty/astc/astc_decompress_symbolic.cpp
    3rdparty/astc/astc_encoding_choice_error.cpp
    3rdparty/astc/astc_find_best_partitioning.cpp
    3rdparty/astc/astc_ideal_endpoints_and_weights.cpp
    3rdparty/astc/astc_imageblock.cpp
    3rdparty/astc/astc_integer_sequence.cpp
    3rdparty/astc/astc_kmeans_partitioning.cpp
    3rdparty/astc/astc_lib.cpp
    3rdparty/astc/astc_partition_tables.cpp
    3rdparty/astc/astc_percentile_tables.cpp
    3rdparty/astc/astc_pick_best_endpoint_format.cpp
    3rdparty/astc/astc_quantization.cpp
    3rdparty/astc/astc_symbolic_physical.cpp
    3rdparty/astc/astc_weight_align.cpp
    3rdparty/astc/astc_weight_quant_xfer_tables.cpp
    3rdparty/astc/mathlib.cpp
    3rdparty/astc/softfloat.cpp
    3rdparty/edtaa3/edtaa3func.cpp
    3rdparty/etc1/etc1.cpp
    3rdparty/etc2/ProcessRGB.cpp
    3rdparty/etc2/Tables.cpp
    3rdparty/iqa/source/convolve.c
    3rdparty/iqa/source/decimate.c
    3rdparty/iqa/source/math_utils.c
    3rdparty/iqa/source/ms_ssim.c
    3rdparty/iqa/source/mse.c
    3rdparty/iqa/source/psnr.c
    3rdparty/iqa/source/ssim.c
    3rdparty/libsquish/alpha.cpp
    3rdparty/libsquish/clusterfit.cpp
    3rdparty/libsquish/colourblock.cpp
    3rdparty/libsquish/colourfit.cpp
    3rdparty/libsquish/colourset.cpp
    3rdparty/libsquish/maths.cpp
    3rdparty/libsquish/rangefit.cpp
    3rdparty/libsquish/singlecolourfit.cpp
    3rdparty/libsquish/squish.cpp
    3rdparty/nvtt/bc6h/zoh.cpp
    3rdparty/nvtt/bc6h/zoh_utils.cpp
    3rdparty/nvtt/bc6h/zohone.cpp
    3rdparty/nvtt/bc6h/zohtwo.cpp
    3rdparty/nvtt/bc7/avpcl.cpp
    3rdparty/nvtt/bc7/avpcl_mode0.cpp
    3rdparty/nvtt/bc7/avpcl_mode1.cpp
    3rdparty/nvtt/bc7/avpcl_mode2.cpp
    3rdparty/nvtt/bc7/avpcl_mode3.cpp
    3rdparty/nvtt/bc7/avpcl_mode4.cpp
    3rdparty/nvtt/bc7/avpcl_mode5.cpp
    3rdparty/nvtt/bc7/avpcl_mode6.cpp
    3rdparty/nvtt/bc7/avpcl_mode7.cpp
    3rdparty/nvtt/bc7/avpcl_utils.cpp
    3rdparty/nvtt/nvmath/fitting.cpp
    3rdparty/nvtt/nvtt.cpp
    3rdparty/pvrtc/BitScale.cpp
    3rdparty/pvrtc/MortonTable.cpp
    3rdparty/pvrtc/PvrTcDecoder.cpp
    3rdparty/pvrtc/PvrTcEncoder.cpp
    3rdparty/pvrtc/PvrTcPacket.cpp
    src/image_cubemap_filter.cpp
    src/image_encode.cpp
)
target_link_libraries(bimg_encode PUBLIC bkaradzic::bx)
target_include_directories(bimg_encode 
    PUBLIC
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    PRIVATE
        3rdparty
        3rdparty/nvtt
        3rdparty/iqa/include
)

add_library(bimg src/image.cpp src/image_gnf.cpp)
target_include_directories(bimg PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include> $<INSTALL_INTERFACE:include>)
target_link_libraries(bimg PUBLIC bkaradzic::bx)

if (BIMG_BUILD_TOOLS)
    add_executable(texturec tools/texturec/texturec.cpp)
    target_link_libraries(texturec PUBLIC bimg_decode bimg_encode bimg)
endif()

################################################################################
# Install configuration
################################################################################

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

install(TARGETS bimg bimg_encode bimg_decode 
    EXPORT ${targets_export_name}
    LIBRARY DESTINATION lib COMPONENT Runtime
    ARCHIVE DESTINATION lib COMPONENT Development
    RUNTIME DESTINATION bin COMPONENT Runtime
    BUNDLE DESTINATION bin COMPONENT Runtime
    PUBLIC_HEADER DESTINATION include COMPONENT Development
    BUNDLE DESTINATION bin COMPONENT Runtime
)

if (BIMG_BUILD_TOOLS)
    install(TARGETS texturec
        EXPORT ${targets_export_name}
        LIBRARY DESTINATION lib COMPONENT Runtime
        ARCHIVE DESTINATION lib COMPONENT Development
        RUNTIME DESTINATION bin COMPONENT Runtime
        BUNDLE DESTINATION bin COMPONENT Runtime
        PUBLIC_HEADER DESTINATION include COMPONENT Development
        BUNDLE DESTINATION bin COMPONENT Runtime
    )
endif()

# Install the generated CMake files.
install(
    EXPORT ${targets_export_name}
    NAMESPACE "bkaradzic::"
    DESTINATION ${ConfigPackageLocation}
)

install(
    FILES
        ${CMAKE_CURRENT_BINARY_DIR}/bimgConfigVersion.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/bimgConfig.cmake
    DESTINATION
        ${ConfigPackageLocation}
)
