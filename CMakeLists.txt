cmake_minimum_required(VERSION 3.21)

include(cmake/prelude.cmake)

project(
    ProtoNet-linux
    VERSION 0.1.0
    DESCRIPTION "Short description"
    HOMEPAGE_URL "https://example.com/"
    LANGUAGES CXX C
)

include(cmake/variables.cmake)

message("Building on: ${CMAKE_SYSTEM_NAME}")

# ---- Declare library ----

file(GLOB_RECURSE secondary_headers source/headers/*/*.h)
file(GLOB main_headers source/headers/*.h)
file(GLOB_RECURSE c_files source/code/*.c)
file(GLOB_RECURSE so_files source/*.so)

message("${secondary_headers}")

add_library(
    ProtoNet-linux_lib OBJECT
    ${main_headers}
    ${secondary_headers}
    ${c_files}
)

target_include_directories(
    ProtoNet-linux_lib ${warning_guard}
    PUBLIC
    "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/source/headers>"
    "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/source/headers/TP/log>"
)

target_compile_features(ProtoNet-linux_lib PUBLIC c_std_17)
if(NOT (${CMAKE_SYSTEM_NAME} MATCHES "Darwin"))
	message("adding libkqueue.so executable")
	target_link_libraries(ProtoNet-linux_lib ${so_files})
endif()
# ---- Declare executable ----

add_executable(ProtoNet-linux_exe source/main.cpp)
add_executable(ProtoNet-linux::exe ALIAS ProtoNet-linux_exe)

set_property(TARGET ProtoNet-linux_exe PROPERTY OUTPUT_NAME ProtoNet-linux)

target_compile_features(ProtoNet-linux_exe PRIVATE cxx_std_17)

target_link_libraries(ProtoNet-linux_exe PRIVATE ProtoNet-linux_lib)

# ---- Install rules ----

if(NOT CMAKE_SKIP_INSTALL_RULES)
  include(cmake/install-rules.cmake)
endif()

# ---- Developer mode ----

if(NOT ProtoNet-linux_DEVELOPER_MODE)
  return()
elseif(NOT PROJECT_IS_TOP_LEVEL)
  message(
      AUTHOR_WARNING
      "Developer mode is intended for developers of ProtoNet-linux"
  )
endif()

include(cmake/dev-mode.cmake)
