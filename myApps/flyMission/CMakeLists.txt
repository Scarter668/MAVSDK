cmake_minimum_required(VERSION 3.15)

# Specify at least C++11, better C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)


# Specify your project's name
project(flyMission)

set(srcname fly_mission_ex)
# Specify your app's executable name, and list of source files used to create it.
add_executable(${srcname}
    ${srcname}.cpp
)

# Finds MAVSDK when installed system wide.
find_package(MAVSDK REQUIRED)

# Specify your app's executable name and a list of linked libraries
target_link_libraries(${srcname}
    MAVSDK::mavsdk             #All apps link against mavsdk library
)

# Enable strict handling of warnings
# add_definitions("-Wall -Wextra -Werror")

if(NOT MSVC)
    add_compile_options(flyMission PRIVATE -Wall -Wextra)
else()
    add_compile_options(flyMission PRIVATE -W2)
endif()

