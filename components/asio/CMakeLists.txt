if(NOT CONFIG_LWIP_IPV6 AND NOT CMAKE_BUILD_EARLY_EXPANSION)
    # note: the component is still included in the build so it can become visible again in config
    # without needing to re-run CMake. However no source or header files are built.
    message(STATUS "IPV6 support is disabled so the asio component will not be built")
    idf_component_register()
    return()
endif()

set(asio_sources "asio/asio/src/asio.cpp")
set(asio_requires lwip pthread)
if(CONFIG_ASIO_SSL_SUPPORT)
    list(APPEND asio_sources
            "port/src/asio_ssl_impl.cpp"
            "port/mbedtls/src/mbedtls_context.cpp"
            "port/mbedtls/src/mbedtls_engine.cpp")
    set(asio_priv_includes "port/mbedtls/include")
list(APPEND asio_requires mbedtls)
endif()

idf_component_register(SRCS ${asio_sources}
                    INCLUDE_DIRS "asio/asio/include" "port/include"
                    PRIV_INCLUDE_DIRS ${asio_priv_includes}
                    REQUIRES ${asio_requires})

# target_compile_options(${COMPONENT_LIB} PRIVATE "-Wno-format")
target_compile_definitions(${COMPONENT_LIB} PUBLIC ASIO_NO_TYPEID)
target_compile_definitions(${COMPONENT_LIB} PUBLIC ASIO_NO_EXCEPTIONS)
target_compile_definitions(${COMPONENT_LIB} PUBLIC SA_RESTART=0x01
                                                   SA_NOCLDSTOP=0x2
                                                   SA_NOCLDWAIT=0x4
)

target_compile_definitions(${COMPONENT_LIB} PUBLIC ASIO_DISABLE_SERIAL_PORT
                                                   ASIO_DISABLE_EVENTFD
                                                   ASIO_SEPARATE_COMPILATION
                                                   ASIO_STANDALONE
                                                   ASIO_HAS_PTHREADS)

# if(CONFIG_ASIO_SSL_SUPPORT)
#     idf_component_get_property(mbedtls mbedtls COMPONENT_LIB)
#     target_link_libraries(${COMPONENT_LIB} PUBLIC ${mbedtls})
# endif()
