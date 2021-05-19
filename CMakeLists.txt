# Copyright 2018 gRPC authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# cmake build file for C++ helloworld example.
# Assumes protobuf and gRPC have been installed using cmake.
# See cmake_externalproject/CMakeLists.txt for all-in-one cmake build
# that automatically builds all the dependencies before building helloworld.

cmake_minimum_required(VERSION 3.13)

project(HelloWorld C CXX)
set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
#set(CMAKE_POSITION_INDEPENDENT_CODE ON)
include(${CMAKE_CURRENT_SOURCE_DIR}/conanbuildinfo.cmake)
include(${CONAN_LIB_DIRS_GRPC}/cmake/grpc_cpp_plugin.cmake)
conan_basic_setup(TARGETS)

find_package(Protobuf REQUIRED)
#find_package(GRPC REQUIRED)
# Proto file
get_filename_component(hw_proto "../../protos/helloworld.proto" ABSOLUTE)
get_filename_component(hw_proto_path "${hw_proto}" PATH)
# Generated sources grpc_cpp_plugin

# Add Library target with protobuf sources

message(STATUS plugins ${CONAN_BIN_DIRS_GRPC})
message(STATUS hw_proto ${hw_proto})
message(STATUS hw_proto_path ${hw_proto_path})
#set(hw_proto_srcs "${CMAKE_CURRENT_BINARY_DIR}/helloworld.pb.cc")
#set(hw_proto_hdrs "${CMAKE_CURRENT_BINARY_DIR}/helloworld.pb.h")
set(hw_grpc_srcs "${CMAKE_CURRENT_BINARY_DIR}/helloworld.grpc.pb.cc")
set(hw_grpc_hdrs "${CMAKE_CURRENT_BINARY_DIR}/helloworld.grpc.pb.h")
set(hw_grpc_proto_PROTOS ${hw_proto})
message(STATUS grpc-plugin ${grpc_cpp_plugin_location})
protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS ${hw_proto})
#grpc_generate_cpp()


message(STATUS ${Protobuf_PROTOC_EXECUTABLE})
add_custom_command(
      OUTPUT "${hw_grpc_srcs}" "${hw_grpc_hdrs}"
      COMMAND ${Protobuf_PROTOC_EXECUTABLE}
      ARGS --grpc_out "${CMAKE_CURRENT_BINARY_DIR}"
        --cpp_out "${CMAKE_CURRENT_BINARY_DIR}"
        -I "${hw_proto_path}"
        --plugin=protoc-gen-grpc="${GRPC_CPP_PLUGIN_PROGRAM}"
        "${hw_proto}"
      DEPENDS "${hw_proto}")

# Include generated *.pb.h files
add_library(hw_grpc_proto ${hw_grpc_srcs} ${hw_grpc_hdrs} ${PROTO_SRCS} ${PROTO_HDRS} )
target_compile_definitions(hw_grpc_proto PUBLIC  _GLIBCXX_USE_CXX11_ABI=0)
#protobuf_generate(TARGET hw_grpc_proto LANGUAGE grpc GENERATE_EXTENSIONS ${hw_grpc_srcs} ${hw_grpc_hdrs} PLUGIN "protoc-gen-grpc="${GRPC_CPP_PLUGIN_PROGRAM})
# hw_grpc_proto

target_link_libraries(hw_grpc_proto
  CONAN_PKG::grpc
  CONAN_PKG::protobuf
  )

# Targets greeter_[async_](client|server)
foreach(_target
  greeter_client greeter_server
  greeter_async_client greeter_async_client2 greeter_async_server)
  add_executable(${_target} "${_target}.cc")
  target_link_libraries(${_target}
    CONAN_PKG::grpc
    CONAN_PKG::protobuf
    hw_grpc_proto
    )
endforeach()
