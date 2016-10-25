cmake_minimum_required (VERSION 3.1.3)

SET(CMAKE_VERBOSE_MAKEFILE ON)
SET(CMAKE_COLOR_MAKEFILE ON)
SET(CMAKE_ERROR_DEPRECATED ON)
SET(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Project Name + Languages - C is here because we require pthreads for testing
project (Arduino CXX C)
# Project description - only used in doxygen and cpack
set(PROJECT_DESCRIPTION "Arduino BSP library based on Arduino/hardware/core")

# Some arbitrary hand-maintained version
set(PROJECT_MAJOR_VERSION 1)
set(PROJECT_MINOR_VERSION 6)
set(PROJECT_PATCH_VERSION 7)
set(PROJECT_VERSION  ${PROJECT_MAJOR_VERSION}.${PROJECT_MINOR_VERSION}.${PROJECT_PATCH_VERSION})

message(STATUS "")
message(STATUS "Running ${CMAKE_COMMAND} ${CMAKE_VERSION} on ${CMAKE_PROJECT_NAME} v${PROJECT_VERSION}" )

# User did not provide install prefix via cmake commandline
if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT )
  SET(CMAKE_INSTALL_PREFIX ${STAGING_DIR} )
endif(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)

# Add definitons without -D!
SET(ARDUINO_COMPILE_DEFINITIONS ARDUINO=${PROJECT_MAJOR_VERSION}${PROJECT_MINOR_VERSION}${PROJECT_PATCH_VERSION})
LIST(APPEND ARDUINO_COMPILE_DEFINITIONS F_CPU=16000000L)
LIST(APPEND ARDUINO_COMPILE_DEFINITIONS USB_VID=null USB_PID=null)

#-------------------------------------------------------------------------------
# Print Summary
#-------------------------------------------------------------------------------
message(STATUS "------------- Config ${PROJECT_NAME} -------------" )
message(STATUS " Build-Type:\t\t ${CMAKE_BUILD_TYPE} " )
message(STATUS " CPU:\t\t ${ARDUINO_MODEL} ")
message(STATUS " Install Prefix:\t ${CMAKE_INSTALL_PREFIX} ")
message(STATUS "" )

#-------------------------------------------------------------------------------
# Input configuration
#-------------------------------------------------------------------------------
# This project has no include / src directory separation
SET(PROJECT_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)
include_directories(${PROJECT_INCLUDE_DIR})

# add . to the includes during the build - redundant here
set(CMAKE_INCLUDE_CURRENT_DIR ON)


#-------------------------------------------------------------------------------
# Output  configuration
#-------------------------------------------------------------------------------

# projectname = name of library (without lib- prefix)
string(TOLOWER "${PROJECT_NAME}" LIBRARY_NAME)
#Since the "standard" cmake template works with component name set it
set(COMPONENT_NAME ${PROJECT_NAME})

# Make debug libs visible per default ->  SET_TARGET_PROPERTIES
SET(CMAKE_DEBUG_POSTFIX "_dbg")
SET(CMAKE_TESTCOVERAGE_POSTFIX "_cov")
# (CMAKE_BINARY_DIR = where cmake was invoked and found project())
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY	"${PROJECT_BINARY_DIR}/bin/")
# Folder for libraries (during build)
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY	"${PROJECT_BINARY_DIR}/lib/")
# Put Archives + Libraries togther
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})


# Offer the user the choice of overriding the installation directories
set(INSTALL_LIB_DIR lib CACHE PATH "Installation directory for libraries")
set(INSTALL_BIN_DIR bin CACHE PATH "Installation directory for executables")
set(INSTALL_INCLUDE_DIR include CACHE PATH  "Installation directory for header files")
set(INSTALL_CMAKE_DIR ${INSTALL_LIB_DIR}/cmake/ CACHE PATH
    "Installation directory for CMake files")
set(INSTALL_DOC_DIR "doc/${PROJECT_NAME}" CACHE PATH  "Installation directory for doxygen docs")

set(generated_dir "${CMAKE_BINARY_DIR}/generated")
set(version_config "${generated_dir}/${COMPONENT_NAME}ConfigVersion.cmake")
set(component_config "${generated_dir}/${COMPONENT_NAME}Config.cmake")
set(targets_export_name "${COMPONENT_NAME}Targets")
export(PACKAGE ${PROJECT_NAME})


#-------------------------------------------------------------------------------
# Include Src to build library libarduino_core.a
#-------------------------------------------------------------------------------
add_subdirectory(src)

# --------------------------------------------------------------------
# Generate CPACK installer
# --------------------------------------------------------------------
set(CPACK_GENERATOR TGZ)
set(CPACK_PACKAGE_NAME ${PROJECT_NAME})
set(CPACK_PACKAGE_VENDOR "Thomas Ruschival")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY ${PROJECT_DESCRIPTION})
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_INSTALL_DIRECTORY ${PROJECT_NAME}-${PROJECT_VERSION})
SET(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_COMPONENT_DEVELOP_DISPLAY_NAME "Libraries + Headers")
set(CPACK_COMPONENT_APIDOC_DISPLAY_NAME "API Documentation")

set(CPACK_COMPONENT_DEVELOP_DESCRIPTION "Components needed for development")
set(CPACK_COMPONENT_APIDOC_DESCRIPTION  "API Documentation")

SET(CPACK_NSIS_CONTACT "t.ruschival@gmail.com")
SET(CPACK_NSIS_MODIFY_PATH ON)

INCLUDE(CPack)

MESSAGE(STATUS "** end CMakeLists.txt **\n")
