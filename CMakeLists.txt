cmake_minimum_required(VERSION 3.15)
project(my_psdk)

set(CMAKE_C_FLAGS "-pthread -std=gnu99")
set(CMAKE_CXX_FLAGS "-std=c++11 -pthread")
set(CMAKE_EXE_LINKER_FLAGS "-pthread")
set(CMAKE_C_COMPILER "aarch64-linux-gnu-gcc")
set(CMAKE_CXX_COMPILER "aarch64-linux-gnu-g++")
add_definitions(-D_GNU_SOURCE)
#-------include path------
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/dji_code/c_module)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/dji_code/c++_module)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/dji_code/hal)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/dji_code/osal)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/dji_code/psdk_lib/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/my_code)
#-------------------------

#-------source path-------
file(GLOB_RECURSE MODULE_SAMPLE_SRC
        ${CMAKE_CURRENT_SOURCE_DIR}/dji_code/c_module/*.c*
        ${CMAKE_CURRENT_SOURCE_DIR}/dji_code/c++_module/*.c*
    )
#file(GLOB_RECURSE MODULE_SAMPLE_SRC )
file(GLOB_RECURSE MODULE_COMMON_SRC ${CMAKE_CURRENT_SOURCE_DIR}/dji_code/osal/*.c*)
file(GLOB_RECURSE MODULE_HAL_SRC ${CMAKE_CURRENT_SOURCE_DIR}/dji_code/hal/*.c*)
file(GLOB_RECURSE MODULE_APP_SRC ${CMAKE_CURRENT_SOURCE_DIR}/my_code/*.c*)
#-------------------------
add_definitions(-DPLATFORM_ARCH_aarch64=1)
add_definitions(-DSYSTEM_ARCH_LINUX=1)
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/dji_code/3rdparty)

#----------link_dir-------
link_directories(${CMAKE_CURRENT_SOURCE_DIR}/dji_code/psdk_lib/lib/aarch64-linux-gnu-gcc)
link_libraries(${CMAKE_CURRENT_SOURCE_DIR}/dji_code/psdk_lib/lib/aarch64-linux-gnu-gcc/libpayloadsdk.a -lstdc++)
add_executable(my_psdk 
    ${MODULE_APP_SRC}
    ${MODULE_SAMPLE_SRC}
    ${MODULE_COMMON_SRC}
    ${MODULE_HAL_SRC}
    )
    find_package(OpenCV QUIET)
if (OpenCV_FOUND)
    message("\n${PROJECT_NAME}...")
    message(STATUS "Found OpenCV installed in the system, will use it to display image in AdvancedSensing APIs")
    message(STATUS " - Includes: ${OpenCV_INCLUDE_DIRS}")
    message(STATUS " - Libraries: ${OpenCV_LIBRARIES}")
    add_definitions(-DOPEN_CV_INSTALLED)
else ()
    message(STATUS "Did not find OpenCV in the system, image data is inside RecvContainer as raw data")
endif ()

find_package(FFMPEG REQUIRED)
if (FFMPEG_FOUND)
    message(STATUS "Found FFMPEG installed in the system")
    message(STATUS " - Includes: ${FFMPEG_INCLUDE_DIR}")
    message(STATUS " - Libraries: ${FFMPEG_LIBRARIES}")

    EXECUTE_PROCESS(COMMAND ffmpeg -version
            OUTPUT_VARIABLE ffmpeg_version_output
            OUTPUT_STRIP_TRAILING_WHITESPACE
            )
    string(REGEX MATCH "version.*Copyright" ffmpeg_version_line ${ffmpeg_version_output})
    string(REGEX MATCH " .* " ffmpeg_version ${ffmpeg_version_line})
    string(REGEX MATCH "^ 5.*$" ffmpeg_major_version ${ffmpeg_version})

    if (HEAD${ffmpeg_major_version} STREQUAL "HEAD")
        message(STATUS " - Version: ${ffmpeg_version}")
    else ()
        message(FATAL_ERROR " - Not support FFMPEG version: ${ffmpeg_major_version}, please install 4.x.x instead.")
    endif ()

    target_link_libraries(my_psdk ${FFMPEG_LIBRARIES})
    include_directories(${FFMPEG_INCLUDE_DIR})
    add_definitions(-DFFMPEG_INSTALLED)
else ()
    message(STATUS "Cannot Find FFMPEG")
endif (FFMPEG_FOUND)




find_package(OPUS REQUIRED)
if (OPUS_FOUND)
    message(STATUS "Found OPUS installed in the system")
    message(STATUS " - Includes: ${OPUS_INCLUDE_DIR}")
    message(STATUS " - Libraries: ${OPUS_LIBRARY}")

    add_definitions(-DOPUS_INSTALLED)
    target_link_libraries(my_psdk /usr/local/lib/libopus.a)
else ()
    message(STATUS "Cannot Find OPUS")
endif (OPUS_FOUND)

find_package(LIBUSB REQUIRED)
if (LIBUSB_FOUND)
    message(STATUS "Found LIBUSB installed in the system")
    message(STATUS " - Includes: ${LIBUSB_INCLUDE_DIR}")
    message(STATUS " - Libraries: ${LIBUSB_LIBRARY}")

    add_definitions(-DLIBUSB_INSTALLED)
    target_link_libraries(my_psdk usb-1.0)
else ()
    message(STATUS "Cannot Find LIBUSB")
endif (LIBUSB_FOUND)
target_link_libraries(${PROJECT_NAME} m)
target_include_directories(${PROJECT_NAME} PRIVATE ${OpenCV_INCLUDE_DIRS})
if (OpenCV_FOUND)
    target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBS})
endif ()
