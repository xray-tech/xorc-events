include_directories(${PROTOBUF_INCLUDE_DIRS})
set(proto-files 
    dmp_event.proto

    common/device.proto
    common/geo.proto
    common/header.proto
    common/browser.proto
    common/carrier.proto
)

protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS 
    ${proto-files}
)

set(cpp_sources)
set(cpp_headers)
set(python_sources)
foreach(proto ${proto-files})
    string(REPLACE ".proto" "" proto-basename ${proto})
    set(root "${CMAKE_CURRENT_BINARY_DIR}/${proto-basename}")
    set(cpp_sources ${cpp_sources} "${root}.pb.cc" "${root}.pb.h")
    set(cpp_headers ${cpp_headers} "${root}.pb.h")
    set(python_sources ${python_sources} "${root}_pb2.py")
    set_source_files_properties("${root}.pb.cc" PROPERTIES GENERATED TRUE)
    set_source_files_properties("${root}.pb.h" PROPERTIES GENERATED TRUE)
    set_source_files_properties("${root}_pb2.py" PROPERTIES GENERATED TRUE)
endforeach()

add_library(event-format
        ${cpp_sources}
    )

target_link_libraries(event-format ${PROTOBUF_LIBRARIES})

target_include_directories(event-format PUBLIC ${CMAKE_CURRENT_BINARY_DIR})

protobuf_generate_python(py ${proto-files})

add_custom_target(generated_code DEPENDS ${PROTO_SRCS} ${PROTO_HDRS} ${py})

# Force the protobuf invocation
add_dependencies(event-format generated_code)

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
