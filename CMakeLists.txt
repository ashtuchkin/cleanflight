cmake_minimum_required(VERSION 3.5)

set(CMAKE_TOOLCHAIN_FILE "arm.toolchain.cmake")
#set(CMAKE_VERBOSE_MAKEFILE TRUE)

project(cleanflight VERSION 1.1.0 LANGUAGES C ASM)

# Main settings
set(TARGET          SPRACINGF3      CACHE STRING "Target board")
set(OPTIONS                         CACHE STRING "Compile-time options")
set(DEBUG_MODE                      CACHE STRING "Debugger optons, must be empty or GDB")
set(SERIAL_DEVICE   "/dev/ttyUSB0"  CACHE STRING "Serial port/Device for flashing")  # $(firstword $(wildcard /dev/ttyUSB*) no-port-found)
set(FLASH_SIZE                      CACHE STRING "Flash size override (KB)")
set(FORKNAME        inav            CACHE STRING "Fork name")

#=================================================================================

# Working directories
set(ROOT        ${CMAKE_SOURCE_DIR})
set(SRC_DIR     ${ROOT}/src/main)
set(LINKER_DIR  ${ROOT}/src/main/target)
set(BIN_DIR     ${ROOT}/obj)

set(CMSIS_DIR   ${ROOT}/lib/main/CMSIS)
set(USBFS_DIR   ${ROOT}/lib/main/STM32_USB-FS-Device_Driver)

include_directories(
    ${SRC_DIR}
    $(ROOT)/src/main/target
)

set(BASE_TARGET ${TARGET})  # TODO: Support derived targets.


set(FC_VER_MAJOR 1)
set(FC_VER_MINOR 2)
set(FC_VER_PATCH 0) # TODO: extract from src/main/version.h

set(FC_VER ${FC_VER_MAJOR}.${FC_VER_MINOR}.${FC_VER_PATCH})


# Output files
set(TARGET_MAP  ${BIN_DIR}/${FORKNAME}_${FC_VER}_${TARGET}.map)
set(TARGET_BIN  ${BIN_DIR}/${FORKNAME}_${FC_VER}_${TARGET}.bin)
set(TARGET_HEX  ${BIN_DIR}/${FORKNAME}_${FC_VER}_${TARGET}.hex)

# ==============================================================

set(TARGET_DIR ${ROOT}/src/main/target/${TARGET})
file(GLOB TARGET_DIR_SRC ${TARGET_DIR}/*.c)
add_definitions(-D${TARGET})

#if (${TARGET} STREQUAL ALIENWIIF1)
#    # ALIENWIIF1 is a VARIANT of NAZE
#    add_definitions(-DNAZE -DALIENWII32)
#    set(TARGET_DIR ${ROOT}/src/main/target/NAZE)
#endif (${TARGET} STREQUAL ALIENWIIF1)

include_directories(${TARGET_DIR})

# ===============================================================

set(COMMON_SRC
        build_config.c
        debug.c
        version.c
        ${TARGET_DIR_SRC}
        main.c
        mw.c
        common/encoding.c
        common/filter.c
        common/maths.c
        common/printf.c
        common/typeconversion.c
        config/config.c
        config/runtime_config.c
        drivers/adc.c
        drivers/buf_writer.c
        drivers/bus_i2c_soft.c
        drivers/bus_spi.c
        drivers/bus_spi_soft.c
        drivers/gps_i2cnav.c
        drivers/gyro_sync.c
        drivers/io.c
        drivers/light_led.c
        drivers/rx_nrf24l01.c
        drivers/rx_xn297.c
        drivers/pwm_mapping.c
        drivers/pwm_output.c
        drivers/pwm_rx.c
        drivers/rcc.c
        drivers/serial.c
        drivers/serial_uart.c
        drivers/sound_beeper.c
        drivers/system.c
        drivers/timer.c
        flight/failsafe.c
        flight/imu.c
        flight/hil.c
        flight/mixer.c
        flight/pid.c
        io/beeper.c
        io/rc_controls.c
        io/rc_curves.c
        io/serial.c
        io/serial_4way.c
        io/serial_4way_avrootloader.c
        io/serial_4way_stk500v2.c
        io/serial_cli.c
        io/serial_msp.c
        io/statusindicator.c
        rx/ibus.c
        rx/msp.c
        rx/nrf24.c
        rx/nrf24_cx10.c
        rx/nrf24_syma.c
        rx/nrf24_v202.c
        rx/nrf24_h8_3d.c
        rx/nrf24_ref.c
        rx/pwm.c
        rx/rx.c
        rx/sbus.c
        rx/spektrum.c
        rx/sumd.c
        rx/sumh.c
        rx/xbus.c
        scheduler/scheduler.c
        scheduler/scheduler_tasks.c
        sensors/acceleration.c
        sensors/battery.c
        sensors/boardalignment.c
        sensors/compass.c
        sensors/gyro.c
        sensors/initialisation.c)

set(HIGHEND_SRC
        blackbox/blackbox.c
        blackbox/blackbox_io.c
        common/colorconversion.c
        drivers/display_ug2864hsweg01.c
        flight/navigation_rewrite.c
        flight/navigation_rewrite_multicopter.c
        flight/navigation_rewrite_fixedwing.c
        flight/navigation_rewrite_pos_estimator.c
        flight/navigation_rewrite_geo.c
        flight/gps_conversion.c
        io/gps.c
        io/gps_ublox.c
        io/gps_nmea.c
        io/gps_naza.c
        io/gps_i2cnav.c
        io/ledstrip.c
        io/display.c
        sensors/rangefinder.c
        sensors/barometer.c
        telemetry/telemetry.c
        telemetry/frsky.c
        telemetry/hott.c
        telemetry/smartport.c
        telemetry/mavlink.c
        telemetry/ltm.c)

# TODO: F4 Targets
set(VCP_SRC
        vcp/hw_config.c
        vcp/stm32_it.c
        vcp/usb_desc.c
        vcp/usb_endp.c
        vcp/usb_istr.c
        vcp/usb_prop.c
        vcp/usb_pwr.c
        drivers/serial_usb_vcp.c)

#====================================================================

set(STM32F10x_COMMON_SRC
        startup/startup_stm32f10x_md_gcc.S
        drivers/adc_stm32f10x.c
        drivers/bus_i2c_stm32f10x.c
        drivers/gpio_stm32f10x.c
        drivers/inverter.c
        drivers/serial_softserial.c
        drivers/serial_uart_stm32f10x.c
        drivers/system_stm32f10x.c
        drivers/timer_stm32f10x.c)

set(STM32F30x_COMMON_SRC
        startup/startup_stm32f30x_md_gcc.S
        target/system_stm32f30x.c
        drivers/adc_stm32f30x.c
        drivers/bus_i2c_stm32f30x.c
        drivers/gpio_stm32f30x.c
        drivers/light_ws2811strip.c
        drivers/light_ws2811strip_stm32f30x.c
        drivers/serial_uart_stm32f30x.c
        drivers/system_stm32f30x.c
        drivers/timer_stm32f30x.c)

set(STM32F4xx_COMMON_SRC
        startup/startup_stm32f40xx.s
        target/system_stm32f4xx.c
        drivers/accgyro_mpu.c
        drivers/adc_stm32f4xx.c
        drivers/adc_stm32f4xx.c
        drivers/bus_i2c_stm32f10x.c
        drivers/gpio_stm32f4xx.c
        drivers/inverter.c
        drivers/serial_softserial.c
        drivers/serial_uart_stm32f4xx.c
        drivers/system_stm32f4xx.c
        drivers/timer_stm32f4xx.c
        drivers/dma_stm32f4xx.c)

# ==================================================================

include(${ROOT}/src/main/target/${TARGET}/CMakeLists.txt)

# ==================================================================

# Calculate FLASH_SIZE
if (FLASH_SIZE)
    # Use given FLASH_SIZE
elseif ("${F1_TARGETS}" MATCHES ${TARGET})
    set(FLASH_SIZE 128)
elseif ("${F3_TARGETS}" MATCHES ${TARGET})
    set(FLASH_SIZE 256)
else (FLASH_SIZE)
    # TODO: F4 targets
    message(ERROR "FLASH_SIZE not configured for target")
endif (FLASH_SIZE)

add_definitions(-DFLASH_SIZE=${FLASH_SIZE})

execute_process(COMMAND git log -1 --format=%h
                WORKING_DIRECTORY ${ROOT}
                OUTPUT_VARIABLE REVISION)



# Board-specific configuration
if ("${F3_TARGETS}" MATCHES ${TARGET})
    # STM32F30x Boards

    set(STDPERIPH_DIR ${ROOT}/lib/main/STM32F30x_StdPeriph_Driver)
    file(GLOB STDPERIPH_SRC ${STDPERIPH_DIR}/src/*.c)
    list(REMOVE_ITEM STDPERIPH_SRC "${STDPERIPH_DIR}/src/stm32f30x_crc.c" "${STDPERIPH_DIR}/src/stm32f30x_can.c")

    # Duplicated entries with STM32F30x_StdPeriph_Driver
    #    file(GLOB CMSIS_SRC
    #            ${CMSIS_DIR}/CM1/CoreSupport/*.c
    #            ${CMSIS_DIR}/CM1/DeviceSupport/ST/STM32F30x/*.c)

    set(TARGET_SRC ${TARGET_SRC} ${STDPERIPH_SRC})

    include_directories(
            ${STDPERIPH_DIR}/inc
            ${CMSIS_DIR}/CM1/CoreSupport
            ${CMSIS_DIR}/CM1/DeviceSupport/ST/STM32F30x)

    # TODO: other boards and features.

    set(LD_SCRIPT stm32_flash_f303_${FLASH_SIZE}k.ld)
    set(ARCH_FLAGS -mthumb -mcpu=cortex-m4 -mfloat-abi=hard -mfpu=fpv4-sp-d16 -fsingle-precision-constant -Wdouble-promotion)
    add_definitions(-DSTM32F303xC -DSTM32F303)

else ("${F1_TARGETS}" MATCHES ${TARGET})
    # STM32F10x Boards: ALIENWIIF1 CC3D CJMCU EUSTM32F103RC NAZE OLIMEXINO PORT103R

    set(STDPERIPH_DIR ${ROOT}/lib/main/STM32F10x_StdPeriph_Driver)
    file(GLOB STDPERIPH_SRC ${STDPERIPH_DIR}/src/*.c)
    list(REMOVE_ITEM STDPERIPH_SRC "${STDPERIPH_DIR}/src/stm32f10x_crc.c" "${STDPERIPH_DIR}/src/stm32f10x_cec.c" "${STDPERIPH_DIR}/src/stm32f10x_can.c")

    set(TARGET_SRC ${TARGET_SRC} ${STDPERIPH_SRC})

#    file(GLOB CMSIS_SRC
#            ${CMSIS_DIR}/CM3/CoreSupport/*.c
#            ${CMSIS_DIR}/CM3/DeviceSupport/ST/STM32F10x/*.c)

    include_directories(
            ${STDPERIPH_DIR}/inc
            ${CMSIS_DIR}/CM3/CoreSupport
            ${CMSIS_DIR}/CM3/DeviceSupport/ST/STM32F10x)

    set(LD_SCRIPT stm32_flash_f103_${FLASH_SIZE}k.ld)
    set(ARCH_FLAGS -mthumb -mcpu=cortex-m3)
    add_compile_options(-pedantic)

#    if (${TARGET} MATCHES "^(EUSTM32F103RC|PORT103R)$")
#        add_definitions(-DSTM32F10X_HD -DSTM32F10X)
#    else (${TARGET} MATCHES "^(EUSTM32F103RC|PORT103R)$")
#        add_definitions(-DSTM32F10X_MD -DSTM32F10X)
#    endif (${TARGET} MATCHES "^(EUSTM32F103RC|PORT103R)$")

    if (${TARGET} STREQUAL CC3D)
        include_directories(${USBFS_DIR}/inc ${ROOT}/src/main/vcp)
        file(GLOB USBPERIPH_SRC "${USBFS_DIR}/src/*.c")
        set(DEVICE_STDPERIPH_SRC ${DEVICE_STDPERIPH_SRC} ${USBPERIPH_SRC})
    endif (${TARGET} STREQUAL CC3D)

endif ("${F3_TARGETS}" MATCHES ${TARGET})


#[[
set(NAZE_SRC
        startup/startup_stm32f10x_md_gcc.S
        drivers/accgyro_adxl345.c
        drivers/accgyro_bma280.c
        drivers/accgyro_l3g4200d.c
        drivers/accgyro_mma845x.c
        drivers/accgyro_mpu.c
        drivers/accgyro_mpu3050.c
        drivers/accgyro_mpu6050.c
        drivers/accgyro_mpu6500.c
        drivers/accgyro_spi_mpu6500.c
        drivers/adc.c
        drivers/adc_stm32f10x.c
        drivers/barometer_bmp085.c
        drivers/barometer_ms5611.c
        drivers/barometer_bmp280.c
        drivers/bus_spi.c
        drivers/bus_i2c_stm32f10x.c
        drivers/compass_hmc5883l.c
        drivers/compass_mag3110.c
        drivers/compass_ak8975.c
        drivers/display_ug2864hsweg01.c
        drivers/flash_m25p16.c
        drivers/gpio_stm32f10x.c
        drivers/inverter.c
        drivers/light_led_stm32f10x.c
        drivers/light_ws2811strip.c
        drivers/light_ws2811strip_stm32f10x.c
        drivers/sonar_hcsr04.c
        drivers/pwm_mapping.c
        drivers/pwm_output.c
        drivers/pwm_rx.c
        drivers/serial_softserial.c
        drivers/serial_uart.c
        drivers/serial_uart_stm32f10x.c
        drivers/sound_beeper_stm32f10x.c
        drivers/system_stm32f10x.c
        drivers/timer.c
        drivers/timer_stm32f10x.c
        io/flashfs.c
        target/NAZE/hardware_revision.c
        ${HIGHEND_SRC}
        ${COMMON_SRC})

set(ALIENWIIF1_SRC ${NAZE_SRC})

set(EUSTM32F103RC_SRC
        startup/startup_stm32f10x_hd_gcc.S
        drivers/accgyro_adxl345.c
        drivers/accgyro_bma280.c
        drivers/accgyro_l3g4200d.c
        drivers/accgyro_mma845x.c
        drivers/accgyro_mpu.c
        drivers/accgyro_mpu3050.c
        drivers/accgyro_mpu6050.c
        drivers/accgyro_spi_mpu6000.c
        drivers/accgyro_spi_mpu6500.c
        drivers/adc.c
        drivers/adc_stm32f10x.c
        drivers/barometer_bmp085.c
        drivers/barometer_ms5611.c
        drivers/bus_i2c_stm32f10x.c
        drivers/bus_spi.c
        drivers/compass_ak8975.c
        drivers/compass_mag3110.c
        drivers/compass_hmc5883l.c
        drivers/display_ug2864hsweg01.c
        drivers/flash_m25p16.c
        drivers/gpio_stm32f10x.c
        drivers/inverter.c
        drivers/light_led_stm32f10x.c
        drivers/light_ws2811strip.c
        drivers/light_ws2811strip_stm32f10x.c
        drivers/pwm_mapping.c
        drivers/pwm_output.c
        drivers/pwm_rx.c
        drivers/serial_softserial.c
        drivers/serial_uart.c
        drivers/serial_uart_stm32f10x.c
        drivers/sonar_hcsr04.c
        drivers/sound_beeper_stm32f10x.c
        drivers/system_stm32f10x.c
        drivers/timer.c
        drivers/timer_stm32f10x.c
        io/flashfs.c
        ${HIGHEND_SRC}
        ${COMMON_SRC})

set(PORT103R_SRC ${EUSTM32F103RC_SRC})

set(OLIMEXINO_SRC
        startup/startup_stm32f10x_md_gcc.S
        drivers/accgyro_mpu.c
        drivers/accgyro_mpu6050.c
        drivers/adc.c
        drivers/adc_stm32f10x.c
        drivers/barometer_bmp085.c
        drivers/bus_i2c_stm32f10x.c
        drivers/bus_spi.c
        drivers/compass_hmc5883l.c
        drivers/compass_mag3110.c
        drivers/compass_ak8975.c
        drivers/gpio_stm32f10x.c
        drivers/light_led_stm32f10x.c
        drivers/light_ws2811strip.c
        drivers/light_ws2811strip_stm32f10x.c
        drivers/pwm_mapping.c
        drivers/pwm_output.c
        drivers/pwm_rx.c
        drivers/serial_softserial.c
        drivers/serial_uart.c
        drivers/serial_uart_stm32f10x.c
        drivers/sonar_hcsr04.c
        drivers/sound_beeper_stm32f10x.c
        drivers/system_stm32f10x.c
        drivers/timer.c
        drivers/timer_stm32f10x.c
        ${HIGHEND_SRC}
        ${COMMON_SRC})

set(CJMCU_SRC
        startup/startup_stm32f10x_md_gcc.S
        drivers/adc.c
        drivers/adc_stm32f10x.c
        drivers/accgyro_mpu.c
        drivers/accgyro_mpu6050.c
        drivers/bus_i2c_stm32f10x.c
        drivers/compass_mag3110.c
        drivers/compass_hmc5883l.c
        drivers/compass_ak8975.c
        drivers/gpio_stm32f10x.c
        drivers/light_led_stm32f10x.c
        drivers/pwm_mapping.c
        drivers/pwm_output.c
        drivers/pwm_rx.c
        drivers/serial_uart.c
        drivers/serial_uart_stm32f10x.c
        drivers/sound_beeper_stm32f10x.c
        drivers/system_stm32f10x.c
        drivers/timer.c
        drivers/timer_stm32f10x.c
        target/CJMCU/hardware_revision.c
        blackbox/blackbox.c
        blackbox/blackbox_io.c
        ${COMMON_SRC})

set(CC3D_SRC
        startup/startup_stm32f10x_md_gcc.S
        drivers/accgyro_mpu.c
        drivers/accgyro_spi_mpu6000.c
        drivers/adc.c
        drivers/adc_stm32f10x.c
        drivers/barometer_bmp085.c
        drivers/barometer_ms5611.c
        drivers/barometer_bmp280.c
        drivers/bus_spi.c
        drivers/bus_i2c_stm32f10x.c
        drivers/compass_mag3110.c
        drivers/compass_hmc5883l.c
        drivers/compass_ak8975.c
        drivers/display_ug2864hsweg01.c
        drivers/flash_m25p16.c
        drivers/gpio_stm32f10x.c
        drivers/inverter.c
        drivers/light_led_stm32f10x.c
        drivers/light_ws2811strip.c
        drivers/light_ws2811strip_stm32f10x.c
        drivers/pwm_mapping.c
        drivers/pwm_output.c
        drivers/pwm_rx.c
        drivers/serial_softserial.c
        drivers/serial_uart.c
        drivers/serial_uart_stm32f10x.c
        drivers/sonar_hcsr04.c
        drivers/sound_beeper_stm32f10x.c
        drivers/system_stm32f10x.c
        drivers/timer.c
        drivers/timer_stm32f10x.c
        io/flashfs.c
        ${HIGHEND_SRC}
        ${COMMON_SRC}
        ${VCP_SRC})

set(NAZE32PRO_SRC 
        ${STM32F30x_COMMON_SRC} 
        ${HIGHEND_SRC} 
        ${COMMON_SRC} 
        ${VCP_SRC})

set(STM32F3DISCOVERY_COMMON_SRC
        ${STM32F30x_COMMON_SRC}
        drivers/accgyro_l3gd20.c
        drivers/accgyro_l3gd20.c
        drivers/accgyro_lsm303dlhc.c
        drivers/compass_mag3110.c
        drivers/compass_hmc5883l.c
        drivers/compass_ak8975.c
        ${VCP_SRC})

set(STM32F3DISCOVERY_SRC
        ${STM32F3DISCOVERY_COMMON_SRC}
        drivers/accgyro_adxl345.c
        drivers/accgyro_bma280.c
        drivers/accgyro_mma845x.c
        drivers/accgyro_mpu.c
        drivers/accgyro_mpu3050.c
        drivers/accgyro_mpu6050.c
        drivers/accgyro_l3g4200d.c
        drivers/barometer_ms5611.c
        drivers/compass_mag3110.c
        drivers/compass_hmc5883l.c
        drivers/compass_ak8975.c
        ${HIGHEND_SRC}
        ${COMMON_SRC})

set(CHEBUZZF3_SRC
        ${STM32F3DISCOVERY_SRC}
        ${HIGHEND_SRC}
        ${COMMON_SRC})

set(COLIBRI_RACE_SRC
        ${STM32F30x_COMMON_SRC}
        drivers/display_ug2864hsweg01.c
        drivers/accgyro_mpu.c
        drivers/accgyro_mpu6500.c
        drivers/accgyro_spi_mpu6500.c
        drivers/accgyro_mpu6500.c
        drivers/barometer_ms5611.c
        drivers/compass_mag3110.c
        drivers/compass_ak8975.c
        drivers/compass_hmc5883l.c
        drivers/serial_usb_vcp.c
        ${HIGHEND_SRC}
        ${COMMON_SRC}
        ${VCP_SRC})

set(LUX_RACE_SRC
        ${STM32F30x_COMMON_SRC}
        drivers/accgyro_mpu.c
        drivers/accgyro_mpu6500.c
        drivers/accgyro_spi_mpu6500.c
        drivers/accgyro_mpu6500.c
        drivers/light_ws2811strip.c
        drivers/light_ws2811strip_stm32f30x.c
        drivers/serial_usb_vcp.c
        ${HIGHEND_SRC}
        ${COMMON_SRC}
        ${VCP_SRC})

set(SPARKY_SRC
        ${STM32F30x_COMMON_SRC}
        drivers/display_ug2864hsweg01.c
        drivers/accgyro_mpu.c
        drivers/accgyro_mpu6050.c
        drivers/barometer_bmp085.c
        drivers/barometer_ms5611.c
        drivers/barometer_bmp280.c
        drivers/compass_mag3110.c
        drivers/compass_ak8975.c
        drivers/compass_hmc5883l.c
        drivers/serial_usb_vcp.c
        ${HIGHEND_SRC}
        ${COMMON_SRC}
        ${VCP_SRC})

set(ALIENWIIF3_SRC ${SPARKY_SRC})

set(RMDO_SRC
        ${STM32F30x_COMMON_SRC}
        drivers/accgyro_mpu.c
        drivers/accgyro_mpu6050.c
        drivers/barometer_bmp085.c
        drivers/barometer_ms5611.c
        drivers/barometer_bmp280.c
        drivers/compass_mag3110.c
        drivers/compass_ak8975.c
        drivers/compass_hmc5883l.c
        drivers/display_ug2864hsweg01.h
        drivers/flash_m25p16.c
        drivers/serial_softserial.c
        drivers/sonar_hcsr04.c
        io/flashfs.c
        ${HIGHEND_SRC}
        ${COMMON_SRC})

set(MOTOLAB_SRC
        ${STM32F30x_COMMON_SRC}
        drivers/accgyro_mpu.c
        drivers/accgyro_mpu6050.c
        drivers/barometer_ms5611.c
        drivers/compass_mag3110.c
        drivers/compass_ak8975.c
        drivers/compass_hmc5883l.c
        drivers/display_ug2864hsweg01.c
        drivers/serial_usb_vcp.c
        drivers/flash_m25p16.c
        io/flashfs.c
        ${HIGHEND_SRC}
        ${COMMON_SRC}
        ${VCP_SRC})
]]
#============================================================

if ("${FEATURES}" MATCHES ONBOARDFLASH)
    set(TARGET_SRC
            ${TARGET_SRC}
            drivers/flash_m25p16.c
            io/flashfs.c)
endif ("${FEATURES}" MATCHES ONBOARDFLASH)


#============================================================

if ("${DEBUG_MODE}" STREQUAL GDB)
    set(OPTIMIZE -O0)
    set(LTO_FLAGS ${OPTIMIZE})
else ("${DEBUG_MODE}" STREQUAL GDB)
    set(OPTIMIZE -Os)
    set(LTO_FLAGS -flto -fuse-linker-plugin ${OPTIMIZE})
endif ("${DEBUG_MODE}" STREQUAL GDB)

set(DEBUG_FLAGS -ggdb3 -DDEBUG)

add_compile_options(
    ${ARCH_FLAGS}
    ${LTO_FLAGS}
    ${DEBUG_FLAGS}
    -std=gnu99
    -Wall -Wextra -Wunsafe-loop-optimizations -Wdouble-promotion
    -ffunction-sections
    -fdata-sections
    -save-temps=obj
    -MMD -MP
)

add_definitions(
    ${OPTIONS}
    -DUSE_STDPERIPH_DRIVER
    -D'__FORKNAME__="${FORKNAME}"'
    -D'__TARGET__="${TARGET}"'
    -D'__REVISION__="${REVISION}"'
)

#======================
string(REPLACE ";" " " ARCH_FLAGS_STR "${ARCH_FLAGS}")
string(REPLACE ";" " " LTO_FLAGS_STR "${LTO_FLAGS}")
string(REPLACE ";" " " DEBUG_FLAGS_STR "${DEBUG_FLAGS}")

set(CMAKE_ASM_FLAGS "${ARCH_FLAGS_STR} -x assembler-with-cpp -MMD -MP" CACHE STRING "Asm flags" FORCE)

set(LINKER_FLAGS "--gc-sections,-Map,${TARGET_MAP},-L${LINKER_DIR}" CACHE STRING "Ld flags")
set(GCC_LINKER_FLAGS "-nostartfiles --specs=nano.specs ${ARCH_FLAGS_STR} ${LTO_FLAGS_STR} ${DEBUG_FLAGS_STR} -static -Wl,${LINKER_FLAGS} -T${LD_SCRIPT}")

set(CMAKE_STATIC_LINKER_FLAGS "${GCC_LINKER_FLAGS}" CACHE STRING "Linker flags" FORCE)
set(CMAKE_MODULE_LINKER_FLAGS "${GCC_LINKER_FLAGS}" CACHE STRING "Linker flags" FORCE)
set(CMAKE_EXE_LINKER_FLAGS    "${GCC_LINKER_FLAGS}" CACHE STRING "Linker flags" FORCE)

#============================================================

# Make source files relative to ${SRC_DIR}
set(SOURCE_FILES_NO_PATH ${TARGET_SRC})
foreach (FILE ${SOURCE_FILES_NO_PATH})
    if (NOT IS_ABSOLUTE ${FILE})
        set(FILE_FULL_PATH "${SRC_DIR}/${FILE}")
        if (NOT EXISTS ${FILE_FULL_PATH})
            message(FATAL_ERROR "File not found: ${FILE}")
        endif (NOT EXISTS ${FILE_FULL_PATH})
        set(SOURCE_FILES ${SOURCE_FILES} ${FILE_FULL_PATH})
    else (NOT IS_ABSOLUTE ${FILE})
        set(SOURCE_FILES ${SOURCE_FILES} ${FILE})
    endif (NOT IS_ABSOLUTE ${FILE})
endforeach (FILE)

# Main elf executable
add_executable(cleanflight "${SOURCE_FILES}")
set_target_properties(cleanflight PROPERTIES SUFFIX ".elf")

#==============================================================

# Generate the hex firmware files that can be flashed to the MCU.
add_custom_command(OUTPUT ${TARGET_HEX}
        COMMAND ${CMAKE_OBJCOPY} -O ihex --set-start 0x8000000 $<TARGET_FILE:cleanflight> ${TARGET_HEX})

add_custom_command(OUTPUT ${TARGET_BIN}
        COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:cleanflight> ${TARGET_BIN})

set(RAM_SIZE 65536) # TODO
add_custom_target(cleanflight_firmware ALL
        COMMENT "Firmware size:"
        COMMAND ${CMAKE_PRINT_SIZE} $<TARGET_FILE_NAME:cleanflight> | tail -1 | xargs bash -c "TEXT=$0; DATA=$1; BSS=$2; TOTAL_FLASH=$((${FLASH_SIZE}*1024)); TOTAL_RAM=${RAM_SIZE}; FLASH=$((TEXT+DATA)); RAM=$((DATA+BSS)); echo \"FLASH: $FLASH ($((FLASH*100/TOTAL_FLASH))%); RAM: $RAM ($((RAM*100/TOTAL_RAM))%); Free RAM: $((TOTAL_RAM-RAM))\";"
        VERBATIM
        DEPENDS ${TARGET_HEX} ${TARGET_BIN})

#==============================================================

add_custom_target(cleanflight_test
        COMMAND ${CMAKE_MAKE_PROGRAM} -C ${CMAKE_SOURCE_DIR}/src/test test)

#==============================================================

# TODO: Cppcheck
# CSOURCES		:= $(shell find $(SRC_DIR) -name '*.c')
# CPPCHECK = cppcheck $(CSOURCES) --enable=all --platform=unix64 --std=c99 --inline-suppr --quiet --force $(addprefix -I,$(INCLUDE_DIRS)) -I/usr/include -I/usr/include/linux

# TODO: Flash
#flash_$(TARGET): $(TARGET_HEX)
#    stty -F $(SERIAL_DEVICE) raw speed 115200 -crtscts cs8 -parenb -cstopb -ixon
#    echo -n 'R' >$(SERIAL_DEVICE)
#    stm32flash -w $(TARGET_HEX) -v -g 0x0 -b 115200 $(SERIAL_DEVICE)

# TODO: Unbrick
#unbrick_$(TARGET): $(TARGET_HEX)
#    stty -F $(SERIAL_DEVICE) raw speed 115200 -crtscts cs8 -parenb -cstopb -ixon
#    stm32flash -w $(TARGET_HEX) -v -g 0x0 -b 115200 $(SERIAL_DEVICE)



