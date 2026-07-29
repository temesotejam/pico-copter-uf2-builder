cmake_minimum_required(VERSION 3.13)

if(NOT DEFINED PICO_SDK_PATH)
    message(FATAL_ERROR "PICO_SDK_PATH is required")
endif()

include("${PICO_SDK_PATH}/external/pico_sdk_import.cmake")

project(pico_copter_cloud_build C CXX ASM)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

pico_sdk_init()

set(
    EIGEN3_INCLUDE_DIR
    "${CMAKE_CURRENT_SOURCE_DIR}/third_party/eigen"
    CACHE PATH
    "Directory containing Eigen/Dense"
)

if(NOT EXISTS "${EIGEN3_INCLUDE_DIR}/Eigen/Dense")
    message(FATAL_ERROR
        "Eigen was not found at ${EIGEN3_INCLUDE_DIR}. "
        "Run the GitHub Actions workflow or provide -DEIGEN3_INCLUDE_DIR=/path/to/eigen."
    )
endif()

set(ORIGINAL_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/source")

add_executable(pico_copter
    "${ORIGINAL_SOURCE_DIR}/pico_copter.cpp"
    "${ORIGINAL_SOURCE_DIR}/sensor.cpp"
    "${ORIGINAL_SOURCE_DIR}/ekf.cpp"
    "${ORIGINAL_SOURCE_DIR}/lsm9ds1_reg.c"
    "${ORIGINAL_SOURCE_DIR}/pwm.cpp"
    "${ORIGINAL_SOURCE_DIR}/radio.cpp"
    "${ORIGINAL_SOURCE_DIR}/control.cpp"
)

target_include_directories(pico_copter PRIVATE
    "${ORIGINAL_SOURCE_DIR}"
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
