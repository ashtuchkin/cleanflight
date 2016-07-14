set(F3_TARGETS ${F3_TARGETS} ${TARGET})
set(FEATURES ONBOARDFLASH)

set(TARGET_SRC
        ${STM32F30x_COMMON_SRC}
        drivers/accgyro_mpu.c
        drivers/accgyro_mpu6050.c
        drivers/barometer_bmp085.c
        drivers/barometer_ms5611.c
        drivers/barometer_bmp280.c
        drivers/compass_mag3110.c
        drivers/compass_ak8975.c
        drivers/compass_hmc5883l.c
        drivers/display_ug2864hsweg01.c
        drivers/flash_m25p16.c
        drivers/serial_softserial.c
        drivers/sonar_hcsr04.c
        io/flashfs.c
        ${HIGHEND_SRC}
        ${COMMON_SRC})
