# Copyright 2020-2021 NVIDIA Corporation.  All rights reserved.
#
# Please refer to the NVIDIA end user license agreement (EULA) associated
# with this source code for terms and conditions that govern your use of
# this software. Any use, reproduction, disclosure, or distribution of
# this software and related documentation outside the terms of the EULA
# is strictly prohibited.

# 3.7 is required for FindVulkan module support in CMake.
cmake_minimum_required(VERSION 3.7)

project(AppDecGL)

if(WIN32)
    set(APP_SOURCES
        ${CMAKE_CURRENT_SOURCE_DIR}/AppDecGL.cpp
    )
    set(NV_FFMPEG_HDRS ${CMAKE_CURRENT_SOURCE_DIR}/../../External/FFmpeg/include)
else ()
    set(APP_SOURCES
        ${CMAKE_CURRENT_SOURCE_DIR}/AppDecGL.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/FramePresenterGLX.cpp
    )
endif()

if(WIN32)
    set(NV_GL_HDRS ${CMAKE_CURRENT_SOURCE_DIR}/../../External/)
else ()
    set(NV_GL_HDRS "")
endif()

set(NV_DEC_SOURCES
 ${NV_DEC_DIR}/NvDecoder.cpp
)

set(NV_ENC_CUDA_UTILS
 ${NVCODEC_UTILS_DIR}/ColorSpace.cu
)

set(NV_DEC_HDRS
 ${NV_DEC_DIR}/NvDecoder.h
 ${NVCODEC_PUBLIC_INTERFACE_DIR}/cuviddec.h
 ${NVCODEC_PUBLIC_INTERFACE_DIR}/nvcuvid.h
 ${NVCODEC_UTILS_DIR}/NvCodecUtils.h
 ${NVCODEC_UTILS_DIR}/FFmpegDemuxer.h
 ${NVCODEC_UTILS_DIR}/ColorSpace.h
 ${NV_APPDEC_COMMON_DIR}/AppDecUtils.h
 ${CMAKE_CURRENT_SOURCE_DIR}/FramePresenter.h
 ${CMAKE_CURRENT_SOURCE_DIR}/FramePresenterGLUT.h
)

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    set(NV_DEC_HDRS ${NV_DEC_HDRS} ${CMAKE_CURRENT_SOURCE_DIR}/FramePresenterGLX.h)
endif()

source_group( "headers" FILES ${NV_DEC_HDRS})
source_group( "sources" FILES ${APP_SOURCES} ${NV_DEC_SOURCES} ${NV_ENC_CUDA_UTILS})

find_package(CUDA)

set(CUDA_HOST_COMPILER ${CMAKE_CXX_COMPILER})

set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};-gencode arch=compute_50,code=\"sm_50,compute_50\")
if ( CMAKE_COMPILER_IS_GNUCC )
    if(NOT "${CUDA_NVCC_FLAGS}" MATCHES "-std=c\\+\\+11" )
        list(APPEND CUDA_NVCC_FLAGS -std=c++11)
    endif()
endif()

cuda_add_executable(${PROJECT_NAME}  ${APP_SOURCES} ${NV_DEC_SOURCES} ${NV_DEC_HDRS} ${NV_ENC_CUDA_UTILS})

set_target_properties(${PROJECT_NAME} PROPERTIES CUDA_SEPARABLE_COMPILATION ON)

target_include_directories(${PROJECT_NAME} PUBLIC ${CUDA_INCLUDE_DIRS}
 ${NVCODEC_PUBLIC_INTERFACE_DIR}
 ${NVCODEC_UTILS_DIR}
 ${NV_CODEC_DIR}
 ${NV_APPDEC_COMMON_DIR}
 ${NV_FFMPEG_HDRS}
 ${NV_GL_HDRS}
)

if(WIN32)
    if(CMAKE_SIZEOF_VOID_P EQUAL 8)
        Set(AVCODEC_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/FFmpeg/lib/x64/avcodec.lib)
        Set(AVFORMAT_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/FFmpeg/lib/x64/avformat.lib)
        Set(AVUTIL_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/FFmpeg/lib/x64/avutil.lib)
        Set(SWRESAMPLE_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/FFmpeg/lib/x64/swresample.lib)
        Set(FREEGLUT_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/GL/lib/x64/freeglut.lib)
        Set(FREEGLUT_STATIC_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/GL/lib/x64/freeglut_static.lib)
        Set(FREEGLUT_STATICD_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/GL/lib/x64/freeglut_staticd.lib)
        Set(FREEGLUTD_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/GL/lib/x64/freeglutd.lib)
        Set(GLEW32_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/GL/lib/x64/glew32.lib)
        Set(GLEW32S_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/GL/lib/x64/glew32s.lib)
        Set(FFMPEG_DLL_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../External/FFmpeg/lib/x64/)
        Set(GL_DLL_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../External/GL/lib/x64/)
     else()
        Set(AVCODEC_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/FFmpeg/lib/Win32/avcodec.lib)
        Set(AVFORMAT_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/FFmpeg/lib/Win32/avformat.lib)
        Set(AVUTIL_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/FFmpeg/lib/Win32/avutil.lib)
        Set(SWRESAMPLE_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/FFmpeg/lib/Win32/swresample.lib)
        Set(FREEGLUT_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/GL/lib/Win32/freeglut.lib)
        Set(FREEGLUT_STATIC_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/GL/lib/Win32/freeglut_static.lib)
        Set(FREEGLUT_STATICD_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/GL/lib/Win32/freeglut_staticd.lib)
        Set(FREEGLUTD_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/GL/lib/Win32/freeglutd.lib)
        Set(GLEW32_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/GL/lib/Win32/glew32.lib)
        Set(GLEW32S_LIB ${CMAKE_CURRENT_SOURCE_DIR}/../../External/GL/lib/Win32/glew32s.lib)
        Set(FFMPEG_DLL_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../External/FFmpeg/lib/Win32/)
        Set(GL_DLL_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../External/GL/lib/Win32/)
    endif()
    #copy dll's
    install(DIRECTORY ${FFMPEG_DLL_DIR} DESTINATION ${NVCODEC_SAMPLES_INSTALL_DIR} FILES_MATCHING PATTERN "*.dll")
    install(DIRECTORY ${GL_DLL_DIR} DESTINATION ${NVCODEC_SAMPLES_INSTALL_DIR} FILES_MATCHING PATTERN "*.dll")
    target_link_libraries(${PROJECT_NAME} ${CUDA_CUDA_LIBRARY} ${CMAKE_DL_LIBS} ${NVENCODEAPI_LIB} ${CUVID_LIB} ${AVCODEC_LIB}
        ${AVFORMAT_LIB} ${AVUTIL_LIB} ${SWRESAMPLE_LIB} ${FREEGLUT_LIB} ${FREEGLUT_STATIC_LIB} ${FREEGLUT_STATICD_LIB}
        ${FREEGLUTD_LIB} ${GLEW32_LIB} ${GLEW32S_LIB})
else ()
    find_library(FREEGLUT_LIB glut)
    find_library(GLEW32_LIB GLEW)
    find_library(X11_LIB X11)
    find_library(GL_LIB GL)
    find_library(CUDART_LIB cudart HINTS ${CUDA_TOOLKIT_ROOT_DIR}/lib64)
    target_link_libraries(${PROJECT_NAME} ${CUDA_CUDA_LIBRARY} ${CUDART_LIB} ${CMAKE_DL_LIBS} ${NVENCODEAPI_LIB} ${CUVID_LIB} ${AVCODEC_LIB}
        ${AVFORMAT_LIB} ${AVUTIL_LIB} ${SWRESAMPLE_LIB} ${FREEGLUT_LIB} ${GLEW32_LIB} ${X11_LIB} ${GL_LIB})
endif()

install(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION ${NVCODEC_SAMPLES_INSTALL_DIR})
if (MSVC)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
    set_target_properties( ${PROJECT_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${NVCODEC_SAMPLES_INSTALL_DIR}/$<CONFIG>/ )
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD  COMMAND ${CMAKE_COMMAND} -E copy_directory ${FFMPEG_DLL_DIR} ${NVCODEC_SAMPLES_INSTALL_DIR}/$<CONFIG>/)
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD  COMMAND ${CMAKE_COMMAND} -E copy_directory ${GL_DLL_DIR} ${NVCODEC_SAMPLES_INSTALL_DIR}/$<CONFIG>/)
    #copy pdb's
    install(FILES $<TARGET_PDB_FILE:${PROJECT_NAME}> DESTINATION ${NVCODEC_SAMPLES_INSTALL_DIR}  CONFIGURATIONS Debug)
endif()

