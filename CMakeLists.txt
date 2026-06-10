cmake_minimum_required(VERSION 3.12)
project(metal-cpp LANGUAGES CXX)

include(GNUInstallDirs)

# Define the library target (compiling AppleFrameworks.cpp to instantiate all implementations)
add_library(metal-cpp AppleFrameworks.cpp)

# Add an alias target with namespace for downstream projects
add_library(metal-cpp::metal-cpp ALIAS metal-cpp)

# Require C++17 as metal-cpp uses constexpr features (e.g. in NS::Object)
target_compile_features(metal-cpp PUBLIC cxx_std_17)

# Specify header search paths for the target (PUBLIC so they propagate to consumers)
target_include_directories(metal-cpp PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

# Private compile options and definitions used when building the library
target_compile_definitions(metal-cpp PRIVATE APPLE_FRAMEWORKS_CPP_INSIDE)
target_compile_options(metal-cpp PRIVATE -Wno-gnu-anonymous-struct)

# Downstream targets and the library itself must link against the required system frameworks (PUBLIC)
find_library(FOUNDATION_FRAMEWORK Foundation REQUIRED)
find_library(METAL_FRAMEWORK Metal REQUIRED)
find_library(QUARTZCORE_FRAMEWORK QuartzCore REQUIRED)
find_library(METALFX_FRAMEWORK MetalFX)
find_library(APPKIT_FRAMEWORK AppKit)
find_library(METALKIT_FRAMEWORK MetalKit)

target_link_libraries(metal-cpp PUBLIC
    ${FOUNDATION_FRAMEWORK}
    ${METAL_FRAMEWORK}
    ${QUARTZCORE_FRAMEWORK}
)

if(METALFX_FRAMEWORK)
    target_link_libraries(metal-cpp PUBLIC ${METALFX_FRAMEWORK})
endif()

if(APPKIT_FRAMEWORK)
    target_link_libraries(metal-cpp PUBLIC ${APPKIT_FRAMEWORK})
endif()

if(METALKIT_FRAMEWORK)
    target_link_libraries(metal-cpp PUBLIC ${METALKIT_FRAMEWORK})
endif()

# --- Installation ---

# Install header directories matching *.hpp files dynamically based on existence
foreach(dir Foundation Metal MetalFX QuartzCore AppKit MetalKit)
    if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${dir}")
        install(DIRECTORY ${dir}
            DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
            FILES_MATCHING PATTERN "*.hpp"
        )
    endif()
endforeach()

# Install top-level master C++ header file if it exists
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/AppleFrameworks.hpp")
    install(FILES AppleFrameworks.hpp DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
endif()

# Install the license file
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.txt")
    install(FILES LICENSE.txt DESTINATION ${CMAKE_INSTALL_DATADIR}/licenses/metal-cpp)
endif()

# Install the metal-cpp target library files
install(TARGETS metal-cpp
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
