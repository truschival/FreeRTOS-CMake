MESSAGE(STATUS "\n Checking ${CMAKE_CURRENT_SOURCE_DIR}")

# Here the component/package = FreeRTOS without subpackages 
SET(COMPONENT_NAME ${PROJECT_NAME})
string(TOLOWER ${COMPONENT_NAME} LIBRARY_NAME)
SET(LIBRARY_VERSION ${PROJECT_VERSION})

#------------------------------
# Dependencies
#------------------------------

# No dependencies so far
SET(MODULE_DEPS "")

#pull in portmacro.h
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/portable/GCC/${PROCESSOR_FAMILY}/)

#------------------------------
# Sources
#------------------------------
set(SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/croutine.c        
  ${CMAKE_CURRENT_SOURCE_DIR}/event_groups.c        
  ${CMAKE_CURRENT_SOURCE_DIR}/list.c        
  ${CMAKE_CURRENT_SOURCE_DIR}/queue.c       
  ${CMAKE_CURRENT_SOURCE_DIR}/tasks.c        
  ${CMAKE_CURRENT_SOURCE_DIR}/timers.c
  #Todo: fix this ugly hack 
  ${CMAKE_CURRENT_SOURCE_DIR}/portable/GCC/${PROCESSOR_FAMILY}/port.c
  # include the specified heap allocator
  ${CMAKE_CURRENT_SOURCE_DIR}/portable/MemMang/heap_${FREERTOS_HEAP_ALLOCATOR}.c
  )

#------------------------------
# Output a library
#------------------------------
ADD_LIBRARY(${LIBRARY_NAME} ${SRCS})

SET_TARGET_PROPERTIES(
  ${LIBRARY_NAME} PROPERTIES
  COMPILE_DEFINITIONS ${PROCESSOR_FAMILY} 
  OUTPUT_NAME ${LIBRARY_NAME}
  VERSION ${LIBRARY_VERSION}
  SOVERSION ${LIBRARY_VERSION}
  )


target_include_directories(
  ${LIBRARY_NAME}
  PRIVATE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
  $<BUILD_INTERFACE:${PROJECT_INCLUDE_DIR}>
  PUBLIC
  $<BUILD_INTERFACE:${PROJECT_INCLUDE_DIR}>
  $<BUILD_INTERFACE:${PROJECT_INCLUDE_DIR}/${COMPONENT_NAME}>
  )

# Add dependencies to library to pass
# dependencies on to client executable
TARGET_LINK_LIBRARIES( ${LIBRARY_NAME}
    ${MODULE_DEPS}
    )


#-----
# Install Targets Library + Headers
#-----
MESSAGE(STATUS "** Generating Package Configurations **")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${version_config}
  VERSION ${LIBRARY_VERSION}
  COMPATIBILITY SameMajorVersion
)

# Configure '<PROJECT-NAME>Config.cmake'
# Note: variable 'targets_export_name' used
configure_file("${CMAKE_SOURCE_DIR}/cmake/Config.cmake.in"
  "${component_config}" @ONLY)

INSTALL(TARGETS ${LIBRARY_NAME}
  EXPORT ${targets_export_name}
  COMPONENT DEVELOP
  ARCHIVE DESTINATION ${INSTALL_LIB_DIR}
  LIBRARY DESTINATION ${INSTALL_LIB_DIR}
  RUNTIME DESTINATION ${INSTALL_BIN_DIR}
  # this will only add -I<prefix>/include to client compile flags
  INCLUDES DESTINATION ${INSTALL_INCLUDE_DIR}/${COMPONENT_NAME}
  )


INSTALL(DIRECTORY
  ${PROJECT_INCLUDE_DIR}/
  COMPONENT DEVELOP
  DESTINATION ${INSTALL_INCLUDE_DIR}/${COMPONENT_NAME}
  )

#install port-specific files
INSTALL(FILES
  ${CMAKE_CURRENT_SOURCE_DIR}/portable/GCC/${PROCESSOR_FAMILY}/portmacro.h
  COMPONENT DEVELOP
  DESTINATION ${INSTALL_INCLUDE_DIR}/${COMPONENT_NAME}
  )

#install configuration
INSTALL(FILES
  ${CMAKE_SOURCE_DIR}/FreeRTOSConfig.h
  COMPONENT DEVELOP
  DESTINATION ${INSTALL_INCLUDE_DIR}/${COMPONENT_NAME}
  )
  
INSTALL(
  EXPORT ${targets_export_name}
  COMPONENT DEVELOP
  DESTINATION "${INSTALL_CMAKE_DIR}"
  )

INSTALL(
  FILES "${component_config}" "${version_config}"
  COMPONENT DEVELOP
  DESTINATION "${INSTALL_CMAKE_DIR}"
  )
 
