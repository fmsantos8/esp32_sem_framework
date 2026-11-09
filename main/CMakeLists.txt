idf_component_register(SRCS "main.c" "application.c" "sht31d.c" "ssd1306.c" "telemetry.c"
    "graphics.c" "fonts.c" "render.c" "bitmaps.c"
    INCLUDE_DIRS "." REQUIRES esp_driver_gpio esp_driver_i2c esp_driver_uart esp_driver_gptimer)
target_compile_features(${COMPONENT_LIB} PRIVATE c_std_11)
target_compile_options(${COMPONENT_LIB} PRIVATE -Wall -Wextra -UNDEBUG)
