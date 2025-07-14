cmake_minimum_required(VERSION 3.28)
project(RISCV-LED C CXX ASM)
# Based on CH32L103

# ——————————————————————————————————————————————
# 1. Toolchain & 基本设置
# ——————————————————————————————————————————————
set(CMAKE_SYSTEM_NAME        Generic)
set(CMAKE_SYSTEM_VERSION     1)
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

# 交叉编译器
set(CMAKE_C_COMPILER         riscv-wch-elf-gcc)
set(CMAKE_CXX_COMPILER       riscv-wch-elf-g++)
set(CMAKE_ASM_COMPILER       riscv-wch-elf-gcc)
set(CMAKE_AR                 riscv-wch-elf-ar)
set(CMAKE_OBJCOPY            riscv-wch-elf-objcopy)
set(CMAKE_OBJDUMP            riscv-wch-elf-objdump)
set(SIZE                    riscv-wch-elf-size)

# 标准
set(CMAKE_C_STANDARD         11)
set(CMAKE_CXX_STANDARD       20)
set(CMAKE_C_STANDARD_REQUIRED   ON)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ——————————————————————————————————————————————
# 2. 公共编译选项
# ——————————————————————————————————————————————
# 架构 & ABI
add_compile_options(
        -march=rv32imacxw
        -mabi=ilp32
        -msmall-data-limit=8
        -msave-restore
)

# 代码质量
add_compile_options(
        -Ofast
        -fmax-errors=20
        -fdump-rtl-expand      # 如果不需要中间展开，可以删掉
        -fmessage-length=0
        -fsigned-char
        -ffunction-sections
        -fdata-sections
        -fno-common
        -Wunused
        -Wuninitialized
        -g                     # 默认带调试信息
)

# ASM 预处理
add_compile_options(
        $<$<COMPILE_LANGUAGE:ASM>:-x>
        $<$<COMPILE_LANGUAGE:ASM>:assembler-with-cpp>
)

# ——————————————————————————————————————————————
# 3. 构建类型相关
# ——————————————————————————————————————————————
# 注意：默认 CMAKE_BUILD_TYPE 可在 cmake 命令行 -DCMAKE_BUILD_TYPE=Release 指定
if(CMAKE_BUILD_TYPE STREQUAL "Release")
    message(STATUS "BuildType=Release → -Ofast")
    # Release 已包含 -Ofast
elseif(CMAKE_BUILD_TYPE STREQUAL "RelWithDebInfo")
    message(STATUS "BuildType=RelWithDebInfo → -Ofast -g")
    # 已包含 -Ofast 和 -g
elseif(CMAKE_BUILD_TYPE STREQUAL "MinSizeRel")
    message(STATUS "BuildType=MinSizeRel → -Os")
    add_compile_options(-Os)
else()
    message(STATUS "BuildType=Debug (或未指定) → -Og -g")
    add_compile_options(-Og -g)
endif()

# ——————————————————————————————————————————————
# 4. include 目录 & 源文件
# ——————————————————————————————————————————————
include_directories(
        ${CMAKE_SOURCE_DIR}/Core
        ${CMAKE_SOURCE_DIR}/Debug
        ${CMAKE_SOURCE_DIR}/Peripheral/inc
        ${CMAKE_SOURCE_DIR}/User
)

file(GLOB_RECURSE
        SRC_CORE      "Core/*.*"
        SRC_DEBUG     "Debug/*.*"
        SRC_PERI      "Peripheral/src/*.*"
        SRC_STARTUP   "Startup/startup_ch32l103.S"
        SRC_USER_C    "User/*.c"
        SRC_USER_CPP  "User/*.cpp"
        SRC_USER_ASM  "User/*.S"
)
set(SOURCES
        ${SRC_CORE}
        ${SRC_DEBUG}
        ${SRC_PERI}
        ${SRC_STARTUP}
        ${SRC_USER_C}
        ${SRC_USER_CPP}
        ${SRC_USER_ASM}
)

# ——————————————————————————————————————————————
# 5. 链接器选项
# ——————————————————————————————————————————————
set(LINKER_SCRIPT ${CMAKE_SOURCE_DIR}/Ld/Link.ld)

# 通用链接选项
add_link_options(
#        -v                              # verbose
        -nostartfiles
        --specs=nano.specs
        --specs=nosys.specs
        -Wl,-gc-sections
        -Wl,--print-memory-usage
        -Wl,-Map=${PROJECT_BINARY_DIR}/${PROJECT_NAME}.map
        -T ${LINKER_SCRIPT}
)

# ——————————————————————————————————————————————
# 6. 生成目标 & 后处理
# ——————————————————————————————————————————————
add_executable(${PROJECT_NAME}.elf
        ${SOURCES}
)

# 生成 .hex 和 .bin
add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -Oihex $<TARGET_FILE:${PROJECT_NAME}.elf> ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex
        COMMAND ${CMAKE_OBJCOPY} -Obinary $<TARGET_FILE:${PROJECT_NAME}.elf> ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin
        COMMENT "Generating ${PROJECT_NAME}.hex and ${PROJECT_NAME}.bin"
)

# ——————————————————————————————————————————————
# 7. 打印 size 信息（可选）
# ——————————————————————————————————————————————
add_custom_target(size_info ALL
        COMMAND ${SIZE} $<TARGET_FILE:${PROJECT_NAME}.elf>
        DEPENDS ${PROJECT_NAME}.elf
        COMMENT "Showing binary size"
)
