add_library(flatccrt
    builder.c
    emitter.c
    refmap.c
    verifier.c
    json_parser.c
    json_printer.c
)

target_include_directories(flatccrt PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>)

if (FLATCC_INSTALL)
    install(TARGETS flatccrt EXPORT flatccTargets DESTINATION ${lib_dir})
endif()
