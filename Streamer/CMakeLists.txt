# If not stated otherwise in this file or this component's LICENSE file the
# following copyright and licenses apply:
#
# Copyright 2020 Metrological
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

project(Streamer)

cmake_minimum_required(VERSION 3.3)

find_package(WPEFramework)

project_version(1.0.0)

set(MODULE_NAME ${NAMESPACE}${PROJECT_NAME})

message("Setup ${MODULE_NAME} v${PROJECT_VERSION}")

set(PLUGIN_STREAMER_AUTOSTART "true" CACHE STRING "Automatically start Streamer plugin")
set(PLUGIN_STREAMER_OUTOFPROCESS "true" CACHE STRING "Controls if the plugin should run in its own process [true, false].")

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

find_package(${NAMESPACE}Definitions REQUIRED)
find_package(${NAMESPACE}Plugins REQUIRED)
find_package(CompileSettingsDebug CONFIG REQUIRED)

add_library(${MODULE_NAME} SHARED
    Module.cpp
    Administrator.cpp
    Streamer.cpp
    StreamerImplementation.cpp
    StreamerJsonRpc.cpp
)

if ("${PLUGIN_STREAMER_IMPLEMENTATIONS}" STREQUAL "")
    message(FATAL_ERROR "No players selected! Select at least one player for the Streamer." )
endif ("${PLUGIN_STREAMER_IMPLEMENTATIONS}" STREQUAL "")

# Make it an array
string(REPLACE " " ";" PLUGIN_STREAMER_IMPLEMENTATIONS ${PLUGIN_STREAMER_IMPLEMENTATIONS})

foreach(IMPL ${PLUGIN_STREAMER_IMPLEMENTATIONS})
    add_subdirectory(Implementation/${IMPL})

    list(APPEND PLAYER_DIRS "Implementation/${IMPL}")
    list(APPEND PLAYER_LIBS "PlayerPlatform${IMPL}")
endforeach(IMPL)

include_directories(${PLAYER_DIRS})

target_include_directories(${MODULE_NAME}
    PRIVATE
        ${CMAKE_CURRENT_LIST_DIR})

set_target_properties(${MODULE_NAME} PROPERTIES
        CXX_STANDARD 11
        CXX_STANDARD_REQUIRED YES)

# "whole-archive" to force linking of the static player libs
target_link_libraries(${MODULE_NAME}
    PRIVATE
        CompileSettingsDebug::CompileSettingsDebug
        ${NAMESPACE}Plugins::${NAMESPACE}Plugins
        ${NAMESPACE}Definitions::${NAMESPACE}Definitions
        -Wl,--whole-archive ${PLAYER_LIBS} -Wl,--no-whole-archive)

install(TARGETS ${MODULE_NAME}
    DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/${STORAGE_DIRECTORY}/plugins)

write_config()
