# Set system and compiler settings
cmake_minimum_required(VERSION 3.22)
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_VERSION 1)

# Project configuration
project(CLionGD32F303CG LANGUAGES C CXX ASM)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_C_STANDARD 11)

# Compiler definitions
set(CMAKE_C_COMPILER arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER arm-none-eabi-g++)
set(CMAKE_ASM_COMPILER arm-none-eabi-gcc)
set(CMAKE_AR arm-none-eabi-ar)
set(CMAKE_OBJCOPY arm-none-eabi-objcopy)
set(CMAKE_OBJDUMP arm-none-eabi-objdump)
set(SIZE arm-none-eabi-size)

# Try compile target type
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

# Set the linker script
set(LINKER_SCRIPT ${CMAKE_SOURCE_DIR}/GD32F303CG_DEFAULT.ld)

# Define project options
add_compile_options(
    -mcpu=cortex-m4
    -mthumb
    -mthumb-interwork
    -ffunction-sections
    -fdata-sections
    -fno-common
    -fmessage-length=0
    -Og
    -g
)

# Specific ASM compile option
add_compile_options($<$<COMPILE_LANGUAGE:ASM>:-x$<SEMICOLON>assembler-with-cpp>)

# Include directories
include_directories(
    Include
    Drivers/GD32F30x_standard_peripheral/Include
    Drivers/CMSIS/
    Drivers/CMSIS/GD/GD32F30x/Include
    Drivers/GD32F30x_usbd_library/class/device/cdc/Include
    Drivers/GD32F30x_usbd_library/device/Include
    Drivers/GD32F30x_usbd_library/usbd/Include
)

# Preprocessor definitions
add_definitions(-DDEBUG -DGD32F3 -DGD32F30x -DGD32F303 -DGD32F30X_XD)

# Source files
file(GLOB_RECURSE SOURCES 
    "Source/*.*" 
    "Drivers/*.*"
)

# Linker options
add_link_options(
    -Wl,-gc-sections
    -Wl,--print-memory-usage
    -Wl,-Map=${PROJECT_BINARY_DIR}/${PROJECT_NAME}.map
    -mcpu=cortex-m4
    -mthumb
    -mthumb-interwork
    -specs=nosys.specs
    -T ${LINKER_SCRIPT}
)

# Executable target
add_executable(${PROJECT_NAME}.elf ${SOURCES})

# Output files (HEX and BIN)
set(HEX_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin)

# Post-build commands
add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -Oihex $<TARGET_FILE:${PROJECT_NAME}.elf> ${HEX_FILE}
    COMMAND ${CMAKE_OBJCOPY} -Obinary $<TARGET_FILE:${PROJECT_NAME}.elf> ${BIN_FILE}
    COMMENT "Generating ${HEX_FILE} and ${BIN_FILE}"
)
