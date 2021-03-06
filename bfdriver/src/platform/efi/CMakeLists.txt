#
# Copyright (C) 2019 Assured Information Security, Inc.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

cmake_minimum_required(VERSION 3.6)
project(efi C CXX)

include(${SOURCE_CMAKE_DIR}/project.cmake)
init_project(
    INCLUDES ../../../include
    INCLUDES ${CMAKE_BINARY_DIR}
)

list(APPEND SOURCES
    entry.c
    platform.c
    platform.asm
    ../../../src/common.c
    ${EFI_EXTENSION_SOURCES}
)

add_custom_command(
    OUTPUT ${CMAKE_BINARY_DIR}/vmm.h
    COMMAND ${CMAKE_COMMAND}
    ARGS -E copy ${BFM_VMM_BIN_PATH}/${BFM_VMM} vmm
    COMMAND xxd
    ARGS -i vmm > ${CMAKE_BINARY_DIR}/vmm.h
    DEPENDS ${BFM_VMM_BIN_PATH}/${BFM_VMM}
    COMMENT "  EFI VMM: ${BFM_VMM_BIN_PATH}/${BFM_VMM}"
)

add_custom_target(
    bareflank_xxd
    DEPENDS ${CMAKE_BINARY_DIR}/vmm.h
)

add_library(bareflank SHARED ${SOURCES})
add_dependencies(bareflank bareflank_xxd)

add_custom_command(
    TARGET bareflank
    POST_BUILD
    COMMAND ${CMAKE_OBJCOPY}
    ARGS -j .text -j .sdata -j .data -j .dynamic -j .dynsym
    ARGS -j .rel -j .rela -j .rel.* -j .rela.* -j .rel* -j .rela* -j .reloc
    ARGS --target efi-app-${BUILD_TARGET_ARCH} libbareflank.so ${CMAKE_BINARY_DIR}/bareflank.efi
)

install(FILES ${CMAKE_BINARY_DIR}/bareflank.efi DESTINATION bin)
