
include_directories(${PROTOBUF_INCLUDE_DIRS})
set(proto-files
    dmp_event.proto
    adjust_event.proto
    routable_message.proto
    sdk_event.proto
    system_event.proto
    backend_event.proto

    http/http_request_event.proto
    http/http_response.proto
    common/device.proto
    common/geo.proto
    common/header.proto
    common/browser.proto
    common/carrier.proto
    common/map_field_entry.proto

    notification/push_notification.proto
    notification/apple_notification.proto
    notification/google_notification.proto
    # Test proto file
    dummy.proto
)

# Generates the matching source files for each proto file
add_custom_target(generated_sources ALL)
add_custom_command(TARGET generated_sources PRE_BUILD COMMAND make -C ${CMAKE_CURRENT_SOURCE_DIR} -f Makefile.sources sources)

set(cpp_sources)
set(cpp_headers)
set(python_sources)
foreach(proto ${proto-files})
    string(REPLACE ".proto" "" proto-basename ${proto})
    set(root "${CMAKE_CURRENT_SOURCE_DIR}/${proto-basename}")
    set(cpp_sources ${cpp_sources} "${root}.pb.cc" "${root}.pb.h")
    set(cpp_headers ${cpp_headers} "${root}.pb.h")
    set(python_sources ${python_sources} "${root}_pb2.py")
    set_source_files_properties("${root}.pb.cc" PROPERTIES GENERATED TRUE)
    set_source_files_properties("${root}.pb.h" PROPERTIES GENERATED TRUE)
    set_source_files_properties("${root}_pb2.py" PROPERTIES GENERATED TRUE)
endforeach()


add_library(event-format ${cpp_sources})
add_dependencies(event-format generated_sources)
target_link_libraries(event-format ${PROTOBUF_LIBRARIES})
target_include_directories(event-format PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

find_package(PythonInterp)
execute_process(COMMAND ${PYTHON_EXECUTABLE} -c "from distutils import sysconfig; print( sysconfig.get_python_lib( plat_specific=True, prefix='${CMAKE_INSTALL_PREFIX}' ) )"
                OUTPUT_VARIABLE _ABS_PYTHON_MODULE_PATH
                OUTPUT_STRIP_TRAILING_WHITESPACE)

get_filename_component(_ABS_PYTHON_MODULE_PATH ${_ABS_PYTHON_MODULE_PATH} ABSOLUTE)
file(RELATIVE_PATH _REL_PYTHON_MODULE_PATH ${CMAKE_INSTALL_PREFIX} ${_ABS_PYTHON_MODULE_PATH})
set(PYTHON_MODULE_PATH ${_REL_PYTHON_MODULE_PATH})

install(FILES ${python_sources}
    DESTINATION ${PYTHON_MODULE_PATH}
    COMPONENT lib)

install(FILES ${cpp_headers}
        DESTINATION include
        COMPONENT includes)
