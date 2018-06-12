# CMake file for building quipper
#------------------------------------------------------------------------------------------------------------------------------------------
# Setup
cmake_minimum_required(VERSION 3.5.1 FATAL_ERROR)
cmake_policy(VERSION 3.5.1)

# - project name
project(quipper LANGUAGES CXX)

# - build type
set(CMAKE_BUILD_TYPE Debug CACHE STRING "Set build type")
set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release")

#------------------------------------------------------------------------------------------------------------------------------------------
# Compiler flags

# - set C++11 flag
if (NOT CMAKE_CXX_FLAGS)
    set(CMAKE_CXX_FLAGS "-std=c++11")
endif()

if("${CMAKE_BUILD_TYPE} " STREQUAL "Debug ")
    set(CMAKE_CXX_FLAGS "-O0 -ggdb ${CMAKE_CXX_FLAGS}")
elseif("${CMAKE_BUILD_TYPE} " STREQUAL "Release ")
    set(CMAKE_CXX_FLAGS "-O3 ${CMAKE_CXX_FLAGS}")
endif()

include(CheckCXXCompilerFlag)
unset(COMPILER_SUPPORTS_CXX_FLAGS CACHE)
CHECK_CXX_COMPILER_FLAG(${CMAKE_CXX_FLAGS} COMPILER_SUPPORTS_CXX_FLAGS)

if(NOT COMPILER_SUPPORTS_CXX_FLAGS)
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} does not support cxx flags ${CMAKE_CXX_FLAGS}")
endif()

#------------------------------------------------------------------------------------------------------------------------------------------
# Build protobuf files

include(FindProtobuf)
find_package(Protobuf REQUIRED)
include_directories(${PROTOBUF_INCLUDE_DIR})
set(PROTOBUF_GENERATE_CPP_APPEND_PATH OFF)

file(GLOB_RECURSE proto_files RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.proto)
protobuf_generate_cpp(PROTO_SRC PROTO_HEADER ${proto_files})

#------------------------------------------------------------------------------------------------------------------------------------------
# Build products

# - build the sources into an object library
set(QUIPPER_SRCS
	src/address_mapper.cc
	src/binary_data_utils.cc
	src/buffer_reader.cc
	src/buffer_writer.cc
	src/conversion_utils.cc
	src/compat/log_level.cc
	src/data_reader.cc
	src/data_writer.cc
	src/dso.cc
	src/file_reader.cc
	src/file_utils.cc
	src/base/logging.cc
	src/perf_option_parser.cc
	src/perf_data_utils.cc
	src/perf_parser.cc
	src/perf_protobuf_io.cc
	src/perf_reader.cc
	src/perf_recorder.cc
	src/perf_serializer.cc
	src/perf_stat_parser.cc
	src/run_command.cc
	src/sample_info_reader.cc
	src/scoped_temp_path.cc
	src/string_utils.cc
	src/huge_page_deducer.cc
	${PROTO_SRC})

set(QUIPPER_OBJECTS quipper-obj)
add_library(${QUIPPER_OBJECTS} OBJECT ${QUIPPER_SRCS})
set_property(TARGET ${QUIPPER_OBJECTS} PROPERTY POSITION_INDEPENDENT_CODE ON)
message(STATUS ${PROTO_HEADER})
target_include_directories(${QUIPPER_OBJECTS} PRIVATE include ${CMAKE_CURRENT_BINARY_DIR})

# - write the static library
set(QUIPPER_STATIC_LIBRARY_NAME quipper-static)
add_library(${QUIPPER_STATIC_LIBRARY_NAME} STATIC $<TARGET_OBJECTS:${QUIPPER_OBJECTS}>)
set_target_properties(${QUIPPER_STATIC_LIBRARY_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME})
target_include_directories(${QUIPPER_STATIC_LIBRARY_NAME} PRIVATE include ${CMAKE_CURRENT_BINARY_DIR})

# - write the shared library
set(QUIPPER_SHARED_LIBRARY_NAME quipper-shared)
add_library(${QUIPPER_SHARED_LIBRARY_NAME} SHARED $<TARGET_OBJECTS:${QUIPPER_OBJECTS}>)
set_target_properties(${QUIPPER_SHARED_LIBRARY_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME})
target_include_directories(${QUIPPER_SHARED_LIBRARY_NAME} PRIVATE include ${CMAKE_CURRENT_BINARY_DIR})

#------------------------------------------------------------------------------------------------------------------------------------------
# Install products

# - directories
install(DIRECTORY DESTINATION ${CMAKE_INSTALL_PREFIX} DIRECTORY_PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
install(DIRECTORY DESTINATION include DIRECTORY_PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
install(DIRECTORY DESTINATION lib DIRECTORY_PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

# - executable
install(TARGETS ${QUIPPER_STATIC_LIBRARY_NAME} ${QUIPPER_SHARED_LIBRARY_NAME} DESTINATION lib PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
install(DIRECTORY include/quipper/ DESTINATION include/quipper FILES_MATCHING PATTERN "*.h" PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
install(FILES ${PROTO_HEADER} DESTINATION include/quipper PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
