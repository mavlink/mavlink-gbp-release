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

# find libraries with cmake modules
find_package(PythonInterp)

# config files
configure_file(config.h.in config.h)
install(FILES ${CMAKE_BINARY_DIR}/config.h DESTINATION include/${PROJECT_NAME} COMPONENT Dev)

# mavlink generation
set(mavgen_path ${CMAKE_SOURCE_DIR}/pymavlink/generator/mavgen.py)
macro(generateMavlink version definitions)
    foreach(definition ${definitions})
        set(targetName ${definition}-v${version})
        set(definitionAbsPath ${CMAKE_SOURCE_DIR}/message_definitions/v${version}/${definition})
        message(STATUS "processing: ${definitionAbsPath}")
        add_custom_command( 
            OUTPUT ${targetName}-stamp
            COMMAND PYTHONPATH=${CMAKE_SOURCE_DIR} ${PYTHON_EXECUTABLE} -m pymavlink.generator.mavgen --lang=C --wire-protocol=${version}
                --output=include/v${version} ${definitionAbsPath}
            COMMAND touch ${targetName}-stamp
            DEPENDS ${definitionAbsPath} ${mavgen_path}
            )
        add_custom_target(${targetName} ALL DEPENDS ${targetName}-stamp)
    endforeach()
endmacro()

# build
set(v0.9Definitions
    ardupilotmega.xml
    common.xml
    minimal.xml
    pixhawk.xml
    slugs.xml
    test.xml
    ualberta.xml
    )
# XXX: generation headers for 0.9 is broken
#generateMavlink("0.9" "${v0.9Definitions}")
set(v1.0Definitions
    ardupilotmega.xml
    autoquad.xml
    common.xml
    matrixpilot.xml
    minimal.xml
    pixhawk.xml
    slugs.xml
    test.xml
    ualberta.xml
    sensesoar.xml
    )
generateMavlink("1.0" "${v1.0Definitions}")

add_subdirectory(pymavlink)

# install files
install(DIRECTORY ${CMAKE_BINARY_DIR}/include/ DESTINATION include/${PROJECT_NAME} COMPONENT Dev FILES_MATCHING PATTERN "*.h*")
install(DIRECTORY ${CMAKE_BINARY_DIR}/src/ DESTINATION share/${PROJECT_NAME} COMPONENT Dev FILES_MATCHING PATTERN "*.c*")
install(DIRECTORY ${CMAKE_SOURCE_DIR}/share/${PROJECT_NAME} DESTINATION share COMPONENT Dev FILES_MATCHING PATTERN "*.c*")

# thanks for urdfdom project
set(PKG_NAME ${PROJECT_NAME})
set(PKG_VERSION ${PROJECT_VERSION})
set(PKG_DESC "mavlink message marshalling library")
set(PKG_LIBRARIES )
set(PKG_DEPENDS )

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

# vim:sw=4:ts=4:expandtab
