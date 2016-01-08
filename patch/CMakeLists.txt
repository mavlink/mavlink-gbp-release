project (mavlink NONE)

# Note: patched version for installation as ROS 3-rd party library
# Provides C-headers and pymavlink

# settings
cmake_minimum_required (VERSION 2.8.2)
set(PROJECT_VERSION_MAJOR "1")
set(PROJECT_VERSION_MINOR "0")
set(PROJECT_VERSION_PATCH "9")
set(PROJECT_VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}")
set(PROJECT_CONTACT_EMAIL http://groups.google.com/group/mavlink)
set(PROJECT_CONTACT_VENDOR mavlink)

include(GNUInstallDirs)

# hack from urdfdom: by default this would be 'lib/x86_64-linux-gnu'
set(CMAKE_INSTALL_LIBDIR lib)

# find libraries with cmake modules
set(PythonInterp_FIND_VERSION "2")
find_package(PythonInterp REQUIRED)

# config files
configure_file(config.h.in config.h)
install(FILES ${CMAKE_BINARY_DIR}/config.h DESTINATION include/${PROJECT_NAME} COMPONENT Dev)

# mavlink generation
set(mavgen_path ${CMAKE_SOURCE_DIR}/pymavlink/tools/mavgen.py)
set(common_xml_path ${CMAKE_SOURCE_DIR}/message_definitions/v1.0/common.xml)
macro(generateMavlink_v10 definitions)
    foreach(definitionAbsPath ${definitions})
        get_filename_component(definition ${definitionAbsPath} NAME_WE)
        message(STATUS "processing: ${definitionAbsPath}")
        add_custom_command(
            OUTPUT include/v1.0/${definition}/${definition}.h
            COMMAND /usr/bin/env PYTHONPATH="${CMAKE_SOURCE_DIR}:$ENV{PYTHONPATH}"
                ${PYTHON_EXECUTABLE} ${mavgen_path} --lang=C --wire-protocol=1.0
                --output=include/v1.0 ${definitionAbsPath}
                DEPENDS ${definitionAbsPath} ${common_xml_path} ${mavgen_path}
            )
        add_custom_target(${definition}.xml-v1.0
            ALL DEPENDS include/v1.0/${definition}/${definition}.h
            )
    endforeach()
endmacro()

# build only v1.0
file(GLOB V10DEFINITIONS ${CMAKE_SOURCE_DIR}/message_definitions/v1.0/*.xml)
generateMavlink_v10("${V10DEFINITIONS}")

# build pymavlink
add_subdirectory(pymavlink)

# install files
install(DIRECTORY ${CMAKE_BINARY_DIR}/include/ DESTINATION include/${PROJECT_NAME} COMPONENT Dev FILES_MATCHING PATTERN "*.h*")
install(DIRECTORY ${CMAKE_BINARY_DIR}/src/ DESTINATION share/${PROJECT_NAME} COMPONENT Dev FILES_MATCHING PATTERN "*.c*")
install(DIRECTORY ${CMAKE_SOURCE_DIR}/share/${PROJECT_NAME} DESTINATION share COMPONENT Dev FILES_MATCHING PATTERN "*.c*")

# thanks for urdfdom project
set(PKG_NAME ${PROJECT_NAME})
set(PKG_VERSION ${PROJECT_VERSION})
set(PKG_DESC "MAVLink micro air vehicle marshalling / communication library")
set(PKG_LIBRARIES )
set(PKG_DEPENDS )
set(PKG_MAVLINK_DEFINITIONS "${V10DEFINITIONS}")
foreach(def ${V10DEFINITIONS})
    get_filename_component(dialect "${def}" NAME_WE)
    list(APPEND PKG_MAVLINK_DIALECTS ${dialect})
endforeach()

configure_file(config.cmake.in ${PROJECT_NAME}-config.cmake @ONLY)
install(FILES ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake/ COMPONENT cmake)

configure_file(pc.in ${PROJECT_NAME}.pc @ONLY)
install(FILES ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig/ COMPONENT pkgconfig)

# add file extensions and set resource files
configure_file("COPYING" "COPYING.txt" COPYONLY)
install(FILES ${PROJECT_BINARY_DIR}/COPYING.txt
    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/ COMPONENT license)

install(FILES ${CMAKE_SOURCE_DIR}/package.xml
    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/ COMPONENT catkin)

# vim:sw=4:ts=4:expandtab
