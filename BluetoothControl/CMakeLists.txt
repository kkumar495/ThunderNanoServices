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
project(BluetoothControl)

cmake_minimum_required(VERSION 3.3)

find_package(WPEFramework)

project_version(1.0.0)

set(MODULE_NAME ${NAMESPACE}${PROJECT_NAME})

message("Setup ${MODULE_NAME} v${PROJECT_VERSION}")

option(PLUGIN_BLUETOOTH_DEVELOPMENT "Enable verbose tracing" ON)
option(PLUGIN_BLUETOOTH_KERNEL_CONNECION_CONTROL "Enable kernel connection control" OFF)

set(PLUGIN_BLUETOOTH_AUTOSTART true CACHE STRING true)

find_package(${NAMESPACE}Bluetooth REQUIRED)
find_package(${NAMESPACE}Plugins REQUIRED)
find_package(${NAMESPACE}Definitions REQUIRED)
find_package(CompileSettingsDebug CONFIG REQUIRED)

add_library(${MODULE_NAME} SHARED
    BluetoothControl.cpp
    BluetoothControlJsonRpc.cpp
    Module.cpp)

set_target_properties(${MODULE_NAME} PROPERTIES
        CXX_STANDARD 11
        CXX_STANDARD_REQUIRED YES)

if(PLUGIN_BLUETOOTH_DEVELOPMENT)
    target_compile_definitions(${MODULE_NAME}
        PRIVATE
            BLUETOOTH_DEVELOPMENT)
endif()

if(PLUGIN_BLUETOOTH_KERNEL_CONNECION_CONTROL)
    target_compile_definitions(${MODULE_NAME}
        PRIVATE
            USE_KERNEL_CONNECTION_CONTROL)
endif()

target_link_libraries(${MODULE_NAME}
    PRIVATE
        CompileSettingsDebug::CompileSettingsDebug
        ${NAMESPACE}Plugins::${NAMESPACE}Plugins
        ${NAMESPACE}Definitions::${NAMESPACE}Definitions
        ${NAMESPACE}Bluetooth::${NAMESPACE}Bluetooth)

install(TARGETS ${MODULE_NAME}
    DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/${STORAGE_DIRECTORY}/plugins)

write_config()
