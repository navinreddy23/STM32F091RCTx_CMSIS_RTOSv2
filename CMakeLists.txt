cmake_minimum_required(VERSION 3.14)

project(nucleo-f091RC)

enable_language(C ASM)
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

set(DRIVERS_DIR ${CMAKE_SOURCE_DIR}/Drivers/STM32F0xx_HAL_Driver)
set(FREERTOS_DIR ${CMAKE_SOURCE_DIR}/Middlewares/Third_Party/FreeRTOS)
set(CMSIS_DIR Drivers/CMSIS)
set(BSP_DIR ${CMAKE_SOURCE_DIR}/BSP/STM32F0xx-Nucleo)
set(STARTUP startup_stm32f091xc.s)

set(BSP_SRC ${BSP_DIR}/stm32f0xx_nucleo.c)
set(CMSIS_OS_SRC ${FREERTOS_DIR}/Source/CMSIS_RTOS_V2/cmsis_os2.c)

file(GLOB STM32F0_DRIVER_SRC ${DRIVERS_DIR}/Src/*.c)
list(FILTER STM32F0_DRIVER_SRC EXCLUDE REGEX ".*template.c$")

file(GLOB SOURCE Src/*.c)

file(GLOB FREERTOS_SOURCE ${FREERTOS_DIR}/Source/*.c)


set(FREERTOS_PORT ${FREERTOS_DIR}/Source/portable/GCC/ARM_CM0/port.c)
set(FREERTOS_HEAP ${FREERTOS_DIR}/Source/portable/MemMang/heap_4.c)

set(EXECUTABLE ${PROJECT_NAME}.out)

add_executable(${EXECUTABLE} ${STARTUP} ${STM32F0_DRIVER_SRC} ${SOURCE} ${FREERTOS_PORT} ${FREERTOS_SOURCE} ${FREERTOS_HEAP} ${CMSIS_OS_SRC})

target_compile_definitions(${EXECUTABLE} PRIVATE
        -DUSE_HAL_DRIVER
        -DSTM32F091xC
        )

target_include_directories(${EXECUTABLE} PRIVATE
        Inc
	${BSP_DIR}
        ${DRIVERS_DIR}/Inc
        ${CMSIS_DIR}/Device/ST/STM32F0xx/Include
        ${CMSIS_DIR}/Include
	${FREERTOS_DIR}/Source/include
	${FREERTOS_DIR}/Source/portable/GCC/ARM_CM0
	${FREERTOS_DIR}/Source/CMSIS_RTOS_V2/
        )

target_compile_options(${EXECUTABLE} PRIVATE
        -mcpu=cortex-m0
        -mthumb
        #-mfpu=fpv4-sp-d16
        #-mfloat-abi=hard

        -fdata-sections
        -ffunction-sections

        -Wall

        $<$<CONFIG:Debug>:-Og>
        )

target_link_options(${EXECUTABLE} PRIVATE
        -T${CMAKE_SOURCE_DIR}/STM32F091RCTx_FLASH.ld
        -mcpu=cortex-m0
        -mthumb
        #-mfpu=fpv4-sp-d16
        #-mfloat-abi=hard
        -specs=nano.specs
        -lc
        -lm
        -lnosys
        -Wl,-Map=${PROJECT_NAME}.map,--cref
        -Wl,--gc-sections
        )

# Print executable size
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND arm-none-eabi-size ${EXECUTABLE})

# Create hex file
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND arm-none-eabi-objcopy -O ihex ${EXECUTABLE} ${PROJECT_NAME}.hex
        COMMAND arm-none-eabi-objcopy -O binary ${EXECUTABLE} ${PROJECT_NAME}.bin)
