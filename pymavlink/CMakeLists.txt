# pymavlink support's python2 only
set(PythonInterp_FIND_VERSION "2")
find_package(PythonInterp REQUIRED)

# part of catkin/python.cmake
set(enable_setuptools_deb_layout OFF)
if(EXISTS "/etc/debian_version")
  set(enable_setuptools_deb_layout ON)
endif()
option(SETUPTOOLS_DEB_LAYOUT "Enable debian style python package layout" ${enable_setuptools_deb_layout})

if(SETUPTOOLS_DEB_LAYOUT)
  message(STATUS "Using Debian Python package layout")
  set(SETUPTOOLS_ARG_EXTRA "--install-layout=deb")
else()
  message(STATUS "Using default Python package layout")
endif()

# prepare installation script
set(INSTALL_SCRIPT ${CMAKE_CURRENT_BINARY_DIR}/python_distutils_install.sh)
configure_file(python_distutils_install.sh.in ${INSTALL_SCRIPT} @ONLY)

install(CODE "execute_process(COMMAND \"${INSTALL_SCRIPT}\" WORKING_DIRECTORY \"${CMAKE_CURRENT_SOURCE_DIR}\")")
