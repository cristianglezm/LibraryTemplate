cmake_minimum_required(VERSION 3.1...3.14)

set(LIBRARY_NAME "LibraryTemplate")
set(LIBRARY_DESCRIPTION "description")

################################################################################
### User options.
################################################################################

option(${LIBRARY_NAME}_BUILD_TESTS "Build Tests" OFF)
option(${LIBRARY_NAME}_BUILD_TOOLS "Build Tools" OFF)
option(${LIBRARY_NAME}_BUILD_EXAMPLES "Build Examples" OFF)
option(${LIBRARY_NAME}_BUILD_STATIC "Build static" OFF)
option(${LIBRARY_NAME}_BUILD_DOCS "Build documentation" OFF)

SET(BUILD_SHARED_LIBS TRUE)
if(${LIBRARY_NAME}_BUILD_STATIC)
	SET(BUILD_SHARED_LIBS FALSE)
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -static-libgcc -static-libstdc++ -static")
endif(${LIBRARY_NAME}_BUILD_STATIC)

################################################################################
### Project files.
################################################################################

project(${LIBRARY_NAME})

SET(CMAKE_INCLUDE_PATH ${CMAKE_SOURCE_DIR})
include_directories("${CMAKE_SOURCE_DIR}/include")
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules/")

FILE(GLOB_RECURSE ${LIBRARY_NAME}_SOURCES "src/*.cpp")
FILE(GLOB_RECURSE ${LIBRARY_NAME}_INCLUDES "include/*.hpp")

if(NOT ${LIBRARY_NAME}_BUILD_STATIC)
	add_library(${LIBRARY_NAME} SHARED ${${LIBRARY_NAME}_SOURCES} ${${LIBRARY_NAME}_INCLUDES})
else()
	add_library(${LIBRARY_NAME} STATIC ${${LIBRARY_NAME}_SOURCES} ${${LIBRARY_NAME}_INCLUDES})
endif(NOT ${LIBRARY_NAME}_BUILD_STATIC)

################################################################################
### Dependencies.
################################################################################

################################################################################
### Enable C++17 and warnings
################################################################################

SET(CMAKE_CXX_STANDARD 17)
SET(CMAKE_CXX_STANDARD_REQUIRED ON)
SET(CMAKE_CXX_EXTENSIONS OFF)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build (Debug or Release)" FORCE)
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g")
elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -fexpensive-optimizations -Wall")
endif()

################################################################################
### Testing
################################################################################

if(${LIBRARY_NAME}_BUILD_TESTS)
    enable_testing()
    add_subdirectory(test)
endif(${LIBRARY_NAME}_BUILD_TESTS)

################################################################################
### examples
################################################################################

if(${LIBRARY_NAME}_BUILD_EXAMPLES)
	add_subdirectory(examples)
endif(${LIBRARY_NAME}_BUILD_EXAMPLES)

################################################################################
### Tools
################################################################################

if(${LIBRARY_NAME}_BUILD_TOOLS)
	add_subdirectory(tools)
endif(${LIBRARY_NAME}_BUILD_TOOLS)

#################################################################################
### Docs
#################################################################################

if(${LIBRARY_NAME}_BUILD_DOCS)
    find_package(Doxygen)
    IF(DOXYGEN_FOUND)
        configure_file(${CMAKE_CURRENT_SOURCE_DIR}/doxyfile.in 
                ${CMAKE_CURRENT_BINARY_DIR}/doxyfile @ONLY)
        SET(DOXYGEN_INPUT "${CMAKE_CURRENT_BINARY_DIR}/doxyfile")
        SET(DOXYGEN_OUTPUT "${CMAKE_SOURCE_DIR}/docs")

        ADD_CUSTOM_COMMAND(
          OUTPUT ${DOXYGEN_OUTPUT}
          COMMAND ${CMAKE_COMMAND} -E echo_append "Building API Documentation..."
          COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_INPUT}
          COMMAND ${CMAKE_COMMAND} -E echo "Done."
          DEPENDS ${DOXYGEN_INPUT})

        ADD_CUSTOM_TARGET(docs ALL DEPENDS ${DOXYGEN_OUTPUT})

        ADD_CUSTOM_TARGET(docs_forced
          COMMAND ${CMAKE_COMMAND} -E echo_append "Building API Documentation..."
          COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_INPUT}
          COMMAND ${CMAKE_COMMAND} -E echo "Done.")
    ENDIF(DOXYGEN_FOUND)
endif(${LIBRARY_NAME}_BUILD_DOCS)

#################################################################################
### Installing Library
#################################################################################

install(TARGETS ${LIBRARY_NAME} LIBRARY DESTINATION lib ARCHIVE DESTINATION .)
install(DIRECTORY include DESTINATION .)

install(FILES LICENSE.md README.md DESTINATION .)