# Generated Cmake Pico project file

cmake_minimum_required(VERSION 3.13)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Initialise pico_sdk from installed location
# (note this can come from environment, CMake cache etc)

# == DO NOT EDIT THE FOLLOWING LINES for the Raspberry Pi Pico VS Code Extension to work ==
if(WIN32)
    set(USERHOME $ENV{USERPROFILE})
else()
    set(USERHOME $ENV{HOME})
endif()
set(sdkVersion 2.2.0)
set(toolchainVersion 14_2_Rel1)
set(picotoolVersion 2.2.0-a4)
set(picoVscode ${USERHOME}/.pico-sdk/cmake/pico-vscode.cmake)
if (EXISTS ${picoVscode})
    include(${picoVscode})
endif()
# ====================================================================================
set(PICO_BOARD pico_w CACHE STRING "Board type")

# Pull in Raspberry Pi Pico SDK (must be before project)
include(pico_sdk_import.cmake)

project(micro-ROS-UDP C CXX ASM)

# Initialise the Raspberry Pi Pico SDK
pico_sdk_init()

# Add executable. Default name is the project name, version 0.1

link_directories(libmicroros)
add_executable(micro-ROS-UDP 
        micro-ROS-UDP.c 
        picow_udp_transports
)

pico_set_program_name(micro-ROS-UDP "micro-ROS-UDP")
pico_set_program_version(micro-ROS-UDP "0.1")

# Modify the below lines to enable/disable output over UART/USB
pico_enable_stdio_uart(micro-ROS-UDP 0)
pico_enable_stdio_usb(micro-ROS-UDP 1)

# Add the standard library to the build
target_link_libraries(micro-ROS-UDP
        microros
        pico_cyw43_arch_lwip_threadsafe_background
        hardware_adc 
        hardware_pwm
        pico_stdlib)

# Add the standard include files to the build
target_include_directories(micro-ROS-UDP PRIVATE
        libmicroros/include
        ${CMAKE_CURRENT_LIST_DIR}
)

# Add any user requested libraries
target_link_libraries(micro-ROS-UDP 
        
        )

pico_add_extra_outputs(micro-ROS-UDP)

