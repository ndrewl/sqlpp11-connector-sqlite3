set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)

find_package(Sqlpp11)

add_library(sqlpp11-connector-sqlite3
		connection.cpp
		bind_result.cpp
		prepared_statement.cpp
		detail/connection_handle.cpp
      )

if (SQLPP_DYNAMIC_LOADING)
    add_library(sqlpp11-connector-sqlite3-dynamic
                    connection.cpp
                    bind_result.cpp
                    prepared_statement.cpp
                    detail/connection_handle.cpp
                    detail/dynamic_libsqlite3.cpp
        )

    get_filename_component(SQLITE3_LIB_FILE ${SQLITE3_LIBRARY} NAME)

    if (WIN32)
        # the import library (.lib) needs to be replaced by .dll
        get_filename_component(SQLITE3_LIB_FILE ${SQLITE3_LIB_FILE} NAME_WE)
        string(APPEND SQLITE3_LIB_FILE ".dll")
    endif()

    set_target_properties(sqlpp11-connector-sqlite3 PROPERTIES DEFINE_SYMBOL SQLPP11_COMPILING_DLL)
    set_target_properties(sqlpp11-connector-sqlite3-dynamic PROPERTIES DEFINE_SYMBOL SQLPP11_COMPILING_DLL)

    target_compile_definitions(sqlpp11-connector-sqlite3-dynamic
                               PUBLIC SQLPP_DYNAMIC_LOADING
                               PRIVATE SQLPP_DYNAMIC_LOADING_FILENAME=${SQLITE3_LIB_FILE})

    target_include_directories(sqlpp11-connector-sqlite3-dynamic
                               PUBLIC
                               ${SQLPP11_INCLUDE_DIR}
                               ${HinnantDate_INCLUDE_DIR}
                                ${SQLITE3_INCLUDE_DIRS}
                               $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
                               $<INSTALL_INTERFACE:include>)
endif()
# TODO Once there is a Sqlpp11 target, add INTERFACE_INCLUDE_DIRECTORIES to sqlpp11
# TODO export Targets for HinnantDate and Sqlite3 as well
target_include_directories(sqlpp11-connector-sqlite3
                           PUBLIC
                           ${SQLITE3_INCLUDE_DIRS}
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../include>
                           $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

if (BUILD_SHARED_LIBS)
    target_compile_definitions(sqlpp11-connector-sqlite3 PUBLIC sqlpp11)
    if (SQLPP_DYNAMIC_LOADING)
        target_compile_definitions(sqlpp11-connector-sqlite3-dynamic PUBLIC sqlpp11)
    endif()
endif()

if (SQLCIPHER)
    target_compile_definitions(sqlpp11-connector-sqlite3 PUBLIC SQLPP_USE_SQLCIPHER)
    target_link_libraries(sqlpp11-connector-sqlite3 SQLCipher::SQLCipher)
    if (SQLPP_DYNAMIC_LOADING)
        target_compile_definitions(sqlpp11-connector-sqlite3-dynamic PUBLIC SQLPP_USE_SQLCIPHER)
        target_include_directories(sqlpp11-connector-sqlite3-dynamic PUBLIC ${SQLCIPHER_INCLUDE_DIRS})
    endif()
else()
    target_link_libraries(sqlpp11-connector-sqlite3 ${SQLITE3_LIBRARIES})
endif()


include(GNUInstallDirs)

install(TARGETS sqlpp11-connector-sqlite3
        EXPORT Sqlpp11ConnectorSqlite3
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(EXPORT Sqlpp11ConnectorSqlite3
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/Sqlpp11ConnectorSqlite3
        NAMESPACE sqlpp11::
        FILE Sqlpp11ConnectorSqlite3Config.cmake)
    
if (SQLPP_DYNAMIC_LOADING)
    install(TARGETS sqlpp11-connector-sqlite3-dynamic
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
endif()
