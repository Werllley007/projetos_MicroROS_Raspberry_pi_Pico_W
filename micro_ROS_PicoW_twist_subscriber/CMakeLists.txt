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

project(micro_ROS_PicoW_twist_subscriber C CXX ASM)

# Initialise the Raspberry Pi Pico SDK
pico_sdk_init()

# Add executable. Default name is the project name, version 0.1
link_directories(libmicroros)
add_executable(micro_ROS_PicoW_twist_subscriber 
        micro_ROS_PicoW_twist_subscriber.c 
        pico_uart_transport.c)

pico_set_program_name(micro_ROS_PicoW_twist_subscriber "micro_ROS_PicoW_twist_subscriber")
pico_set_program_version(micro_ROS_PicoW_twist_subscriber "0.1")

# Modify the below lines to enable/disable output over UART/USB
pico_enable_stdio_uart(micro_ROS_PicoW_twist_subscriber 0)
pico_enable_stdio_usb(micro_ROS_PicoW_twist_subscriber 1)
add_compile_definitions(PICO_UART_ENABLE_CRLF_SUPPORT=0)
add_compile_definitions(PICO_STDIO_ENABLE_CRLF_SUPPORT=0)
add_compile_definitions(PICO_STDIO_DEFAULT_CRLF=0)


# Add the standard library to the build
target_link_libraries(micro_ROS_PicoW_twist_subscriber
        pico_stdlib
        microros
)

# Add the standard include files to the build
target_include_directories(micro_ROS_PicoW_twist_subscriber PRIVATE
        ${CMAKE_CURRENT_LIST_DIR}
        ${CMAKE_CURRENT_LIST_DIR}/libmicroros/include
)

# Add any user requested libraries
target_link_libraries(micro_ROS_PicoW_twist_subscriber 
        
        )

pico_add_extra_outputs(micro_ROS_PicoW_twist_subscriber)

