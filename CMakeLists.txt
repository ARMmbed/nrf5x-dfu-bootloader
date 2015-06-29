#
# CMake configuration
#
# Please refer to http://www.cmake.org/cmake/help/documentation.html
# You may also refer to http://www.cmake.org/cmake/help/syntax.html for a quick
# introduction to CMake's syntax.

cmake_minimum_required (VERSION 2.8)

# The name of our project is "BLE_BOOTLOADER". CMakeLists files in this project can
# refer to the root source directory of the project as ${BLE_BOOTLOADER_SOURCE_DIR}
# and to the root binary directory of the project as ${BLE_BOOTLOADER_BINARY_DIR}.
project (BLE_BOOTLOADER)

# define some more paths to projects we depend on
set(MBED_SRC_PATH       ${BLE_BOOTLOADER_SOURCE_DIR}/mbed/libraries/mbed/
    CACHE DIRECTORY     "MBED SDK path")
set(BLE_API_SRC_PATH    ${BLE_BOOTLOADER_SOURCE_DIR}/BLE_API/
    CACHE DIRECTORY     "BLE API path")
set(NRF51822_SRC_PATH   ${BLE_BOOTLOADER_SOURCE_DIR}/nRF51822/
    CACHE DIRECTORY     "nRF51822 API path")
set(APP_PATH            ${BLE_BOOTLOADER_SOURCE_DIR}/DefaultApp.hex
    CACHE FILE          "Default application to bundle with the bootloader")

set(SOFT_DEVICE		${MBED_SRC_PATH}/targets/hal/TARGET_NORDIC/TARGET_MCU_NRF51822/Lib/s130_nrf51822_1_0_0/s130_nrf51_1.0.0_softdevice.hex)

set(TOOLCHAIN_SYSROOT   /usr/local/
    CACHE DIRECTORY     "Path to an ARM toolchain")

# It's best to hide all the details of setting up the variable SRCS in a CMake
# macro. The macro can then be called in all the project CMake list files to add
# sources.
#
# The macro first computes the path of the source file relative to the project
# root for each argument. If the macro is invoked from inside a project sub
# directory the new value of the variable SRCS needs to be propagated to the
# parent folder by using the PARENT_SCOPE option.
#
# Source: http://stackoverflow.com/questions/7046956/populating-srcs-from-cmakelists-txt-in-subdirectories
macro (add_sources)
    file (RELATIVE_PATH _relPath "${CMAKE_SOURCE_DIR}" "${CMAKE_CURRENT_SOURCE_DIR}")
    foreach (_src ${ARGN})
        if (_relPath)
            list (APPEND SRCS "${_relPath}/${_src}")
        else()
            list (APPEND SRCS "${_src}")
        endif()
    endforeach()
    if (_relPath)
        # propagate to parent directory
        set (SRCS ${SRCS} PARENT_SCOPE)
    endif()
endmacro()

# decide about the actual compilers to be used ...
set(CMAKE_CXX_COMPILER ${TOOLCHAIN_SYSROOT}/bin/armcc)
set(CMAKE_C_COMPILER   ${TOOLCHAIN_SYSROOT}/bin/armcc)
set(LINKER             ${TOOLCHAIN_SYSROOT}/bin/armlink)
set(SIZE_COMMAND       size)
set(MAIN_TARGET        ${PROJECT_NAME}.elf)
set(HEX_TARGET         ${PROJECT_NAME}.hex)
set(COMBINED_SD        "combined.hex"
    CACHE FILE         "Output name for the bundled bootloader+softdevice")
set(COMBINED_SD_APP    "combined_app.hex"
    CACHE FILE         "Output name for the bundled bootloader+softdevice+app")

set(CMAKE_LINK_FLAGS   "--libpath=${TOOLCHAIN_SYSROOT}/lib --info=totals --list=.link_totals.txt --scatter ${BLE_BOOTLOADER_SOURCE_DIR}/bootloader.sct")

set(CMAKE_CXX_LINK_EXECUTABLE
    "${LINKER} ${CMAKE_LINK_FLAGS} <OBJECTS> -o <TARGET> --map --feedback=.feedback --feedback_type=unused,iw")
set(CMAKE_C_LINK_EXECUTABLE
    "${LINKER} ${CMAKE_LINK_FLAGS} <OBJECTS> -o <TARGET> --map --feedback=.feedback --feedback_type=unused,iw")
enable_language(ASM)

message(STATUS "C compiler  : ${CMAKE_C_COMPILER}")
message(STATUS "C++ compiler: ${CMAKE_CXX_COMPILER}")
message(STATUS "Size command: ${SIZE_COMMAND}")
message(STATUS "Main target : ${MAIN_TARGET}")
message(STATUS "HEX target  : ${HEX_TARGET}")
message(STATUS "Combined    : ${COMBINED_SD}")
message(STATUS "Combined+app: ${COMBINED_SD_APP}")

############################################################################
# Build type should be clear from here so we
# can continue with selecting include directors, defines
# and other compiler/linker flags ...
############################################################################
# include directories
include_directories(
    ${BLE_BOOTLOADER_SOURCE_DIR}
    ${BLE_BOOTLOADER_SOURCE_DIR}/include
    ${MBED_SRC_PATH}/
    ${MBED_SRC_PATH}/api
    ${MBED_SRC_PATH}/common
    ${MBED_SRC_PATH}/hal
    ${MBED_SRC_PATH}/targets
    ${MBED_SRC_PATH}/targets/cmsis
    ${MBED_SRC_PATH}/targets/cmsis/TARGET_NORDIC
    ${MBED_SRC_PATH}/targets/cmsis/TARGET_NORDIC/TARGET_MCU_NRF51822
    ${MBED_SRC_PATH}/targets/cmsis/TARGET_NORDIC/TARGET_MCU_NRF51822/TOOLCHAIN_ARM_STD
    ${MBED_SRC_PATH}/targets/hal
    ${MBED_SRC_PATH}/targets/hal/TARGET_NORDIC
    ${MBED_SRC_PATH}/targets/hal/TARGET_NORDIC/TARGET_MCU_NRF51822
    ${MBED_SRC_PATH}/targets/hal/TARGET_NORDIC/TARGET_MCU_NRF51822/Lib
    ${MBED_SRC_PATH}/targets/hal/TARGET_NORDIC/TARGET_MCU_NRF51822/Lib/nordic_sdk/components/libraries/crc16
    ${MBED_SRC_PATH}/targets/hal/TARGET_NORDIC/TARGET_MCU_NRF51822/Lib/nordic_sdk/components/libraries/scheduler
    ${MBED_SRC_PATH}/targets/hal/TARGET_NORDIC/TARGET_MCU_NRF51822/Lib/nordic_sdk/components/libraries/util
    ${MBED_SRC_PATH}/targets/hal/TARGET_NORDIC/TARGET_MCU_NRF51822/TARGET_NRF51822_MKIT
    ${BLE_API_SRC_PATH}
    ${BLE_API_SRC_PATH}/public
    ${BLE_API_SRC_PATH}/common
    ${BLE_API_SRC_PATH}/services
    ${NRF51822_SRC_PATH}
    ${NRF51822_SRC_PATH}/btle
    ${NRF51822_SRC_PATH}/btle/custom
    ${NRF51822_SRC_PATH}/common
    ${NRF51822_SRC_PATH}/nordic
    ${NRF51822_SRC_PATH}/nordic-sdk/components/
    ${NRF51822_SRC_PATH}/nordic-sdk/components/ble
    ${NRF51822_SRC_PATH}/nordic-sdk/components/ble/ble_services/ble_dfu
    ${NRF51822_SRC_PATH}/nordic-sdk/components/ble/ble_radio_notification
    ${NRF51822_SRC_PATH}/nordic-sdk/components/ble/common
    ${NRF51822_SRC_PATH}/nordic-sdk/components/ble/device_manager
    ${NRF51822_SRC_PATH}/nordic-sdk/components/ble/device_manager/config
    ${NRF51822_SRC_PATH}/nordic-sdk/components/drivers_nrf
    ${NRF51822_SRC_PATH}/nordic-sdk/components/drivers_nrf/ble_flash
    ${NRF51822_SRC_PATH}/nordic-sdk/components/drivers_nrf/hal
    ${NRF51822_SRC_PATH}/nordic-sdk/components/drivers_nrf/pstorage
    ${NRF51822_SRC_PATH}/nordic-sdk/components/drivers_nrf/pstorage/config
    ${NRF51822_SRC_PATH}/nordic-sdk/components/libraries
    ${NRF51822_SRC_PATH}/nordic-sdk/components/libraries/bootloader_dfu
    ${NRF51822_SRC_PATH}/nordic-sdk/components/libraries/bootloader_dfu/hci_transport
    ${NRF51822_SRC_PATH}/nordic-sdk/components/libraries/crc16
    ${NRF51822_SRC_PATH}/nordic-sdk/components/libraries/gpiote
    ${NRF51822_SRC_PATH}/nordic-sdk/components/libraries/hci
    ${NRF51822_SRC_PATH}/nordic-sdk/components/libraries/scheduler
    ${NRF51822_SRC_PATH}/nordic-sdk/components/libraries/util
    ${NRF51822_SRC_PATH}/nordic-sdk/components/softdevice
    ${NRF51822_SRC_PATH}/nordic-sdk/components/softdevice/common
    ${NRF51822_SRC_PATH}/nordic-sdk/components/softdevice/common/softdevice_handler
    ${NRF51822_SRC_PATH}/nordic-sdk/components/softdevice/s130
    ${NRF51822_SRC_PATH}/nordic-sdk/components/softdevice/s130/include
)

# Generic compiler flags
add_definitions(
    --cpu=Cortex-M0
    --apcs=interwork
    --enum_is_int
    -DNRF51
    -DDEBUG_NRF_USER
    -DBOARD_NRF6310
    -DBOOTLOADER_BANKED
    # -DBLE_STACK_SUPPORT_REQD
    -DUSE_APP_TIMER
    -O3
    --md
    -DTARGET_NRF51822
    -DTARGET_M0
    -DTARGET_NORDIC
    -DTOOLCHAIN_ARM_STD
    -DTOOLCHAIN_ARM
    -D__CC_ARM
    -D__CORTEX_M0
    -DARM_MATH_CM0
    -D__MBED__=1
    -DNEED_PSTORAGE
    -DNEED_APP_GPIOTE
    --feedback=.feedback
)

# Language specifc compiler flags.
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --cpp --no_rtti")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} --c99")

# A macro to collect local sources into ${SRCS}.
# This variable gets propagated to the parent scope and is ultimately used in
# the top-level CMakeLists.txt to define the dependencies for the build target.
#
# Please note that files within this list are relative to the current folder.
# Please also note that this macro must be used at all CMakeLists.txt files at
# intermediate levels even if the list is empty--this is due to the Cmake magic
# involved in propagating variables to only the parent scope.
add_sources(
    main.c
    dfu_dual_bank.c
    bootloader.c
    bootloader_settings_arm.c
    dfu_transport_ble.c
    dfu_ble_svc.c
    app_timer.c
    app_timer_appsh.c
)

add_sources(${BLE_BOOTLOADER_SOURCE_DIR}/startup/arm_startup_nrf51.s)
add_sources(${MBED_SRC_PATH}/targets/cmsis/TARGET_NORDIC/TARGET_MCU_NRF51822/system_nrf51.c)

file(GLOB NRF51822_SOURCES
          ${NRF51822_SRC_PATH}/btle/custom/*.cpp
          ${NRF51822_SRC_PATH}/nordic-sdk/components/ble/ble_services/ble_dfu/*.c
          ${NRF51822_SRC_PATH}/nordic-sdk/components/ble/common/*.c
          ${NRF51822_SRC_PATH}/nordic-sdk/components/ble/common/*.cpp
          ${NRF51822_SRC_PATH}/nordic-sdk/components/ble/device_manager/*.c
          ${NRF51822_SRC_PATH}/nordic-sdk/components/drivers_nrf/hal/*.c
          ${NRF51822_SRC_PATH}/nordic-sdk/components/drivers_nrf/pstorage/*.c
          ${NRF51822_SRC_PATH}/nordic-sdk/components/libraries/bootloader_dfu/*.c
          ${NRF51822_SRC_PATH}/nordic-sdk/components/libraries/bootloader_dfu/experimental/*.c
          ${NRF51822_SRC_PATH}/nordic-sdk/components/libraries/crc16/*.c
          ${NRF51822_SRC_PATH}/nordic-sdk/components/libraries/hci/*.c
          ${NRF51822_SRC_PATH}/nordic-sdk/components/libraries/scheduler/*.c
          ${NRF51822_SRC_PATH}/nordic-sdk/components/libraries/util/*.c
          ${NRF51822_SRC_PATH}/nordic-sdk/components/softdevice/common/softdevice_handler/*.c
    )
add_sources(${NRF51822_SOURCES})

############################################################################
# By now, we've traversed all subdirectories and have collected everything that
# needs to be built. We can define the build targets.
############################################################################
# add MbedTest as a build target depending on all the sources
add_executable(${MAIN_TARGET} ${SRCS})

# Add a post-build dependency like printing size of the
# resulting binary and copying to the target.
add_custom_command(
    # Show ELF size
    TARGET ${MAIN_TARGET}
    COMMAND ${SIZE_COMMAND} ${MAIN_TARGET}
)

add_custom_target(${HEX_TARGET} ALL
    DEPENDS ${MAIN_TARGET}
    COMMAND ${TOOLCHAIN_SYSROOT}/bin/fromelf --i32combined -o ${PROJECT_NAME}.hex ${MAIN_TARGET}
)

add_custom_target(${COMBINED_SD} ALL
    COMMENT "Add a SoftDevice to the HEX package"
    DEPENDS ${HEX_TARGET}
    COMMAND srec_cat ${SOFT_DEVICE} -intel
        ${HEX_TARGET} -intel
        -o ${COMBINED_SD} -intel -obs=16
)

add_custom_target(${COMBINED_SD_APP} ALL
    COMMENT "Add an application to the HEX package"
    DEPENDS ${COMBINED_SD}
    COMMAND srec_cat ${COMBINED_SD} -intel
        ${APP_PATH} -intel

        # Change bootloader settings: BANK0 now contains a valid application
        -exclude 0x3FC00 0x3FC20
        -generate 0x3FC00 0x3FC04 -l-e-constant 0x01 4
        -generate 0x3FC04 0x3FC08 -l-e-constant 0x00 4
        -generate 0x3FC08 0x3FC0C -l-e-constant 0xFE 4
        -generate 0x3FC0C 0x3FC20 -constant 0x00
        -o ${COMBINED_SD_APP} -intel -obs=16
    COMMAND srec_info ${COMBINED_SD_APP} -intel
    # follow this by copying the resulting combined.hex onto the target (possibly over USB)
)
