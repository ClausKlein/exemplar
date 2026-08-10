# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception

cmake_minimum_required(VERSION 3.30...4.4)

include(../infra/cmake/enable-experimental-import-std.cmake)

project(beman.exemplar.examples LANGUAGES CXX)

if(PROJECT_IS_TOP_LEVEL)
    find_package(beman.exemplar 2.4.1 EXACT REQUIRED)
    enable_testing()
endif()

set(ALL_EXAMPLES identity_direct_usage)

# Example `identity_as_default_projection` need ranges support:
include(CheckCXXSymbolExists)
check_cxx_symbol_exists(__cpp_lib_ranges "ranges" HAS_RANGES)

if(HAS_RANGES)
    list(APPEND ALL_EXAMPLES identity_as_default_projection)
else()
    message(
        WARNING
        "Missing range support! Skip: identity_as_default_projection"
    )
endif()

message("Examples to be built: ${ALL_EXAMPLES}")

foreach(example ${ALL_EXAMPLES})
    add_executable(beman.exemplar.examples.${example})
    target_sources(beman.exemplar.examples.${example} PRIVATE ${example}.cpp)
    target_link_libraries(
        beman.exemplar.examples.${example}
        PRIVATE beman::exemplar
    )
    if(BEMAN_USE_STD_MODULE OR BEMAN_EXEMPLAR_USE_MODULES)
        set_target_properties(
            beman.exemplar.examples.${example}
            PROPERTIES CXX_MODULE_STD ON
        )
    endif()
    if(BEMAN_USE_STD_MODULE)
        target_compile_definitions(
            beman.exemplar.examples.${example}
            PRIVATE BEMAN_USE_STD_MODULE
        )
    endif()
    add_test(
        NAME beman.exemplar.examples.${example}
        COMMAND beman.exemplar.examples.${example}
    )
endforeach()
