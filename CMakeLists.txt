cmake_minimum_required(VERSION 3.15)
project(xrobotoolkit_sdk LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(PYBIND11_FINDPYTHON ON)
find_package(pybind11 CONFIG REQUIRED)

# --- locate the prebuilt native PXREARobotSDK ---------------------------------
# By default use the sibling XRoboToolkit-PC-Service submodule (checked out next
# to this one under third_party/). Override with -DXRT_NATIVE_LIB_DIR=... .
cmake_host_system_information(RESULT ISA_NAME QUERY OS_PLATFORM)
if(ISA_NAME STREQUAL "aarch64")
  set(_xrt_plat "linux_aarch64")
else()
  set(_xrt_plat "linux")
endif()

if(NOT DEFINED XRT_NATIVE_LIB_DIR)
  set(XRT_NATIVE_LIB_DIR
      "${CMAKE_CURRENT_SOURCE_DIR}/../XRoboToolkit-PC-Service/RoboticsService/SDK/${_xrt_plat}/64")
endif()
get_filename_component(XRT_NATIVE_LIB_DIR "${XRT_NATIVE_LIB_DIR}" ABSOLUTE)
message(STATUS "XRT_NATIVE_LIB_DIR = ${XRT_NATIVE_LIB_DIR}")

if(NOT EXISTS "${XRT_NATIVE_LIB_DIR}/libPXREARobotSDK.so")
  message(FATAL_ERROR
      "libPXREARobotSDK.so not found in ${XRT_NATIVE_LIB_DIR}. Check out the "
      "XRoboToolkit-PC-Service submodule (git submodule update --init --recursive) "
      "or pass -DXRT_NATIVE_LIB_DIR=<dir>.")
endif()

# --- build the binding --------------------------------------------------------
pybind11_add_module(xrobotoolkit_sdk MODULE bindings/py_bindings.cpp)
target_include_directories(xrobotoolkit_sdk PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/include")
target_link_directories(xrobotoolkit_sdk PRIVATE "${XRT_NATIVE_LIB_DIR}")
target_link_libraries(xrobotoolkit_sdk PRIVATE PXREARobotSDK)

# Find libPXREARobotSDK.so at runtime: next to the module ($ORIGIN, for the
# bundled wheel copy) and at its fixed submodule location (for editable installs).
set_target_properties(xrobotoolkit_sdk PROPERTIES
    BUILD_WITH_INSTALL_RPATH TRUE
    INSTALL_RPATH "$ORIGIN;${XRT_NATIVE_LIB_DIR}")

install(TARGETS xrobotoolkit_sdk LIBRARY DESTINATION .)
install(FILES "${XRT_NATIVE_LIB_DIR}/libPXREARobotSDK.so" DESTINATION .)
