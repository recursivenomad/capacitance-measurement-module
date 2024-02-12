# The versions below are what Raspberry Pi uses, no explicit reasoning provided
cmake_minimum_required(VERSION 3.13)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

# We are using the Raspberry Pi Pico (rather than the Pico W or other boards)
set(PICO_BOARD pico CACHE STRING "Board type")

# Tell the Pico SDK to not optimize anything when we're debugging
set(PICO_DEOPTIMIZED_DEBUG ON)

# Include the Pico SDK (various build functions and compiling definitions)
include($ENV{PICO_SDK_PATH}/external/pico_sdk_import.cmake)

# TODO - Set this to the lowest version required for the features being used
if (PICO_SDK_VERSION_STRING VERSION_LESS "1.5.1")
  message(FATAL_ERROR "Raspberry Pi Pico SDK version 1.5.1 (or later) required. Your version is ${PICO_SDK_VERSION_STRING}")
endif()

# Set PROJECT_NAME and languages
project(main C CXX ASM)

# Invokes the Pico SDK's CMakeLists.txt (and its children); and
# Creates a pico-sdk subdirectory in our project for the libraries
pico_sdk_init()



# Tell CMake where/what our executable source file is
add_executable(${PROJECT_NAME}
    main.c
)

# Link the project to the defined libraries for compilation
target_link_libraries(${PROJECT_NAME}
    pico_stdlib  # Defined in .../pico-sdk/src/rp2_common/pico_stdlib/CMakeLists.txt
)



# Sets firmware metadata, as visible by using picotool
pico_set_program_name(${PROJECT_NAME}
    "")
pico_set_program_version(${PROJECT_NAME}
    "")
pico_set_program_description(${PROJECT_NAME}
    "")
pico_set_program_url(${PROJECT_NAME}
    "")

# Toggles for USB/UART stdio output
pico_enable_stdio_usb(${PROJECT_NAME} OFF)
pico_enable_stdio_uart(${PROJECT_NAME} ON)

# Creates map/bin/hex/uf2 files, in addition to ELF
pico_add_extra_outputs(${PROJECT_NAME})
