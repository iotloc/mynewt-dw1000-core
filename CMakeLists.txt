cmake_minimum_required(VERSION 3.7)
project(dw1000-core VERSION ${VERSION} LANGUAGES C)
set(CMAKE_BUILD_TYPE ${BUILD_TYPE})

set(CMAKE_EXPORT_PACKAGE_REGISTRY ON)

configure_file (
  "${PROJECT_SOURCE_DIR}/config.h.in"
  "${PROJECT_BINARY_DIR}/../include/config.h"
  )

# Use GNUInstallDirs to install libraries into correct
# locations on all platforms.
include(GNUInstallDirs)

add_subdirectory(apps/syscfg)
add_subdirectory(porting/dpl)
add_subdirectory(porting/dpl_os)
add_subdirectory(hw/drivers/uwb/uwb_dw1000)
add_subdirectory(lib)

if (CMAKE_BUILD_TYPE MATCHES "^[Rr]elease")
find_package(Doxygen)
if(DOXYGEN_FOUND)
	option(BUILD_DOC "Build documentation" ON)
	set(DOXYGEN_IN ${CMAKE_CURRENT_SOURCE_DIR}/doxy/Doxyfile-dw1000.conf)
    	set(DOXYGEN_OUT ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)
	configure_file(${DOXYGEN_IN} ${DOXYGEN_OUT} @ONLY)
    	message("Doxygen build started")

	add_custom_target(doc_doxygen ALL
        COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_OUT}
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        COMMENT "Generating API documentation with Doxygen"
        VERBATIM )
endif()
endif()

# build a CPack driven installer package
include (InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_GENERATOR "STGZ;TGZ;TZ")
set(CPACK_COMPONENTS_ALL applications libraries headers)
set(CPACK_COMPONENT_APPLICATIONS_DISPLAY_NAME "dw1000-core")
set(CPACK_COMPONENT_APPLICATIONS_DESCRIPTION "Decawave DW1000 driver")
set(CPACK_COMPONENT_LIBRARIES_DESCRIPTION "Static libraries used to build programs with example")
set(CPACK_COMPONENT_HEADERS_DESCRIPTION "C/C++ header files for use with template")
set(CPACK_COMPONENT_DOCS_DISPLAY_NAME "C++ API Documentation")
set(CPACK_COMPONENT_DOCS_DESCRIPTION "C++ API HTML files describing the API for Interface")
set(CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME}-${VERSION}-${GIT_SHORT_HASH}-${CMAKE_SYSTEM_PROCESSOR}" )

if (CMAKE_BUILD_TYPE MATCHES "^[Rr]elease")
find_package(Doxygen)
if(DOXYGEN_FOUND)
install (DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/html DESTINATION ./)
endif()
endif()

include(CPack)
include(CTest)

# does the application run
add_test (TutorialRuns Tutorial 25)

