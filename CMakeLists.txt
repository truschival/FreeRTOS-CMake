cmake_minimum_required (VERSION 3.3.0)

SET(CMAKE_VERBOSE_MAKEFILE ON)
SET(CMAKE_COLOR_MAKEFILE ON)
SET(CMAKE_ERROR_DEPRECATED ON)
SET(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Project Name + Languages 
project (FreeRTOS C)
# Project description - only used in doxygen and cpack
set(PROJECT_DESCRIPTION "FreeRTOS as CMake package")

# Some arbitrary hand-maintained version
set(PROJECT_MAJOR_VERSION 9)
set(PROJECT_MINOR_VERSION 0)
set(PROJECT_PATCH_VERSION 0)
set(PROJECT_VERSION  ${PROJECT_MAJOR_VERSION}.${PROJECT_MINOR_VERSION}.${PROJECT_PATCH_VERSION})

message(STATUS "")
message(STATUS "Running ${CMAKE_COMMAND} ${CMAKE_VERSION} on ${CMAKE_PROJECT_NAME} v${PROJECT_VERSION}" )

# User did not provide install prefix via cmake commandline
if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT )
  SET(CMAKE_INSTALL_PREFIX ${STAGING_DIR} )
endif(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)

#-------------------------------------------------------------------------------
# Print Summary
#-------------------------------------------------------------------------------
message(STATUS "------------- Config ${PROJECT_NAME} -------------" )
message(STATUS " Build-Type:\t\t ${CMAKE_BUILD_TYPE} " )
message(STATUS " CPU Family:\t\t ${PROCESSOR_FAMILY} ")
message(STATUS " CPU:\t\t ${TIVA_PART_NAME} ")
message(STATUS " Install Prefix:\t ${CMAKE_INSTALL_PREFIX} ")
message(STATUS "" )

#-------------------------------------------------------------------------------
# Input configuration
#-------------------------------------------------------------------------------
SET(PROJECT_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)

#FreeRTOS offers some configuration options
IF(NOT FREERTOS_HEAP_ALLOCATOR)
    SET(FREERTOS_HEAP_ALLOCATOR 2
        CACHE INT "FreeRTOS memory allocator"
        FORCE)
ENDIF(NOT FREERTOS_HEAP_ALLOCATOR)

# Find other configuration options in FreeRTOSConfig.h
include_directories(${CMAKE_CURRENT_SOURCE_DIR})


#-------------------------------------------------------------------------------
# Output  configuration
#-------------------------------------------------------------------------------

# projectname = name of library (without lib- prefix)
string(TOLOWER "${PROJECT_NAME}" LIBRARY_NAME)
#Since the "standard" cmake template works with component name set it
set(COMPONENT_NAME ${PROJECT_NAME})

# Make debug libs visible per default ->  SET_TARGET_PROPERTIES
SET(CMAKE_DEBUG_POSTFIX "_dbg")
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
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(component_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
export(PACKAGE ${PROJECT_NAME})


#-------------------------------------------------------------------------------
# Include Src to build library driverlib, utils and drivers
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


