add_library(mavsdk)


# --------------------------------------------------------------------------
# In src/mavsdk/CMakeLists.txt, right after add_library(mavsdk)
target_include_directories(mavsdk
    PUBLIC
        # used while building from source
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        # used after "make install" (superbuild stages headers under include/)
        $<INSTALL_INTERFACE:include>
)

# --- Build-time include alias so #include <mavsdk/plugins/...> resolves ---
# We create a virtual include root under <build>/gen_include/mavsdk/plugins
# and symlink each plugin's public headers into it.
set(GEN_INCLUDE_DIR ${CMAKE_BINARY_DIR}/gen_include)
file(MAKE_DIRECTORY ${GEN_INCLUDE_DIR}/mavsdk/plugins)

# Find all plugin public include roots:
#   .../mavsdk/plugins/<plugin>/include/plugins/<plugin>
file(GLOB PLUGIN_PUBLIC_DIRS
     "${PROJECT_SOURCE_DIR}/mavsdk/plugins/*/include/plugins/*")

# For each public include dir, create:
#   <build>/gen_include/mavsdk/plugins/<plugin> -> <source>/.../include/plugins/<plugin>
foreach(pubdir IN LISTS PLUGIN_PUBLIC_DIRS)
  get_filename_component(plugin_name "${pubdir}" NAME)
  set(dest "${GEN_INCLUDE_DIR}/mavsdk/plugins/${plugin_name}")
  execute_process(
    COMMAND ${CMAKE_COMMAND} -E create_symlink "${pubdir}" "${dest}"
  )
endforeach()

# Use that alias only while building from source.
target_include_directories(mavsdk
    PUBLIC
        $<BUILD_INTERFACE:${GEN_INCLUDE_DIR}>
)
# --------------------------------------------------------------------------


cmake_policy(SET CMP0076 NEW)

add_subdirectory(core)
add_subdirectory(plugins)

hunter_add_package(jsoncpp)
find_package(jsoncpp REQUIRED)

hunter_add_package(tinyxml2)
find_package(tinyxml2 REQUIRED)

target_link_libraries(mavsdk
    PRIVATE
    JsonCpp::JsonCpp
    tinyxml2::tinyxml2
)

if (NOT APPLE AND NOT ANDROID AND NOT MSVC)
    target_link_libraries(mavsdk
        PRIVATE
        stdc++fs
    )
endif()

set(UNIT_TEST_SOURCES ${UNIT_TEST_SOURCES} PARENT_SCOPE)
