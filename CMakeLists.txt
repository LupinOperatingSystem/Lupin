cmake_minimum_required(VERSION 3.16)
project(Lupin C CXX ASM)

set(CMAKE_INSTALL_MESSAGE LAZY)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    add_compile_options(-fconcepts -Wno-literal-suffix)
endif()

unset(CMAKE_SYSROOT)
set(CMAKE_STAGING_PREFIX ${CMAKE_BINARY_DIR}/Root)
set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/Root)

if (NOT ${CMAKE_HOST_SYSTEM_NAME} MATCHES "Lupin")
    set(TOOLCHAIN_PATH /home/krasno/opt/cross/bin)
    set(TOOLCHAIN_PREFIX ${TOOLCHAIN_PATH}/x86_64-elf-)

    set(CMAKE_C_COMPILER ${TOOLCHAIN_PREFIX}gcc)
    set(CMAKE_CXX_COMPILER ${TOOLCHAIN_PREFIX}g++)
    set(CMAKE_ASM_COMPILER ${TOOLCHAIN_PREFIX}gcc)
    set(CMAKE_LINKER ${TOOLCHAIN_PREFIX}ld)
    set(CMAKE_RANLIB ${TOOLCHAIN_PREFIX}ranlib)
    set(CMAKE_STRIP ${TOOLCHAIN_PREFIX}strip)
    set(CMAKE_AR ${TOOLCHAIN_PREFIX}ar)
endif()

foreach(lang ASM C CXX OBJC OBJCXX)
    unset(CMAKE_${lang}_OSX_COMPATIBILITY_VERSION_FLAG)
    unset(CMAKE_${lang}_OSX_CURRENT_VERSION_FLAG)
    unset(CMAKE_${lang}_LINK_FLAGS)
    unset(CMAKE_SHARED_LIBRARY_CREATE_${lang}_FLAGS)
    unset(CMAKE_SHARED_MODULE_CREATE_${lang}_FLAGS)
    unset(CMAKE_SHARED_MODULE_LOADER_${lang}_FLAG )
    unset(CMAKE_${lang}_OSX_DEPLOYMENT_TARGET_FLAG)
    unset(CMAKE_${lang}_SYSROOT_FLAG)
    if (CMAKE_SYSTEM_NAME MATCHES Darwin)
        ## MacOS Workaround. Don't generate install_name flag when cross compiling
        set(CMAKE_${lang}_CREATE_SHARED_LIBRARY
            "<CMAKE_${lang}_COMPILER> <LANGUAGE_COMPILE_FLAGS> <CMAKE_SHARED_LIBRARY_CREATE_${lang}_FLAGS> <LINK_FLAGS> -o <TARGET> <OBJECTS> <LINK_LIBRARIES>")
    endif()
endforeach()

set(CMAKE_INSTALL_NAME_TOOL "")
set(CMAKE_SHARED_LIBRARY_SUFFIX ".so")
set(CMAKE_SHARED_LIBRARY_CREATE_CXX_FLAGS "-shared -Wl,--hash-style=gnu,-z,relro,-z,now,-z,noexecstack")
set(CMAKE_CXX_LINK_FLAGS "-Wl,--hash-style=gnu,-z,relro,-z,now,-z,noexecstack")

add_compile_options(-fdiagnostics-color=always)

add_compile_options(-fno-exceptions)
add_compile_options(-O2)

add_link_options(--sysroot ${CMAKE_BINARY_DIR}/Root)

add_subdirectory(Kernel)
