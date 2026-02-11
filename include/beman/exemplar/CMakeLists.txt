# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception

target_sources(
    beman.exemplar_headers
    PUBLIC FILE_SET HEADERS FILES identity.hpp
)

if(BEMAN_USE_MODULES)
    target_sources(
        beman.exemplar
        PUBLIC FILE_SET CXX_MODULES FILES identity.cppm
    )
    # NOTE: Require C++20 or newer for modules is needed here!
    # ORIG: target_compile_features(beman.exemplar PUBLIC cxx_std_${CMAKE_CXX_STANDARD})
    target_compile_features(beman.exemplar PUBLIC cxx_std_23)
    target_link_libraries(beman.exemplar PUBLIC beman::exemplar_headers)
    if(WIN32 AND EXEMPLAR_BUILD_SHARED)
        set_target_properties(
            beman.exemplar
            PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS ON
        )
    endif()
endif()

set_target_properties(
    beman.exemplar_headers
    PROPERTIES VERIFY_INTERFACE_HEADER_SETS ON
)
