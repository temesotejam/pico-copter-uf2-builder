cmake_minimum_required(VERSION 3.13)

if(NOT DEFINED PICO_SDK_PATH)
    if(DEFINED ENV{PICO_SDK_PATH})
        set(PICO_SDK_PATH "$ENV{PICO_SDK_PATH}")
    else()
        message(FATAL_ERROR "PICO_SDK_PATH is required")
    endif()
endif()

include("${PICO_SDK_PATH}/external/pico_sdk_import.cmake")

project(pico_copter C CXX ASM)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

pico_sdk_init()

set(EIGEN3_INCLUDE_DIR "" CACHE PATH "Directory containing Eigen/Dense")

if(NOT EIGEN3_INCLUDE_DIR)
    include(FetchContent)
    FetchContent_Declare(
        eigen
        GIT_REPOSITORY https://gitlab.com/libeigen/eigen.git
        GIT_TAG 3147391d946bb4b6c68edd901f2add6ac1f31f8c
    )
    FetchContent_GetProperties(eigen)
    if(NOT eigen_POPULATED)
        FetchContent_Populate(eigen)
    endif()
    set(EIGEN3_INCLUDE_DIR "${eigen_SOURCE_DIR}")
endif()

if(NOT EXISTS "${EIGEN3_INCLUDE_DIR}/Eigen/Dense")
    message(FATAL_ERROR "Eigen/Dense was not found at ${EIGEN3_INCLUDE_DIR}")
endif()

add_executable(pico_copter
    pico_copter.cpp
    sensor.cpp
    ekf.cpp
    lsm9ds1_reg.c
    pwm.cpp
    radio.cpp
    control.cpp
)

target_include_directories(pico_copter PRIVATE
    "${CMAKE_CURRENT_SOURCE_DIR}"
    "${EIGEN3_INCLUDE_DIR}"
)

target_link_libraries(pico_copter PRIVATE
    hardware_spi
    hardware_pwm
    hardware_uart
    pico_stdlib
    pico_multicore
)

pico_enable_stdio_usb(pico_copter 1)
pico_enable_stdio_uart(pico_copter 0)
pico_add_extra_outputs(pico_copter)
