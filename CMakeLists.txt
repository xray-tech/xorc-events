project(events C CXX)
cmake_minimum_required(VERSION 3.0)


set(CMAKE_MODULE_PATH
    ${events_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

find_package(Protobuf REQUIRED)

add_subdirectory(schema)


set(CPACK_GENERATOR "TGZ")
set(CPACK_COMPONENTS_ALL binaries libraries includes configuration tools)
include(CPack)
