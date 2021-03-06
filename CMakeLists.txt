cmake_minimum_required( VERSION 2.8 )

project( pyreclab )

set( MAJOR_VERSION "0" )
set( MINOR_VERSION "1" )
set( RELEASE_VERSION "7" )
set( PACKAGE_VERSION "${MAJOR_VERSION}.${MINOR_VERSION}.${RELEASE_VERSION}" )

# Set CMake macros
SET( CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH}
                       ${PROJECT_SOURCE_DIR}/cmake_scripts )

# Python version
set( CMAKE_PYTHON_VERSION 2.7 CACHE STRING "Python version" )
find_package( PythonLibs "${CMAKE_PYTHON_VERSION}" REQUIRED )

# Compiler definitions
add_definitions( -DNDEBUG )

add_compile_options( -fPIC )
add_compile_options( -O3 )

# header directories
include_directories( ${PROJECT_SOURCE_DIR} )
include_directories( dataio )
include_directories( datahandlers )
include_directories( algorithms )
include_directories( eval_metrics )

# External paths
include_directories( ${PYTHON_INCLUDE_DIR} )

add_subdirectory( dataio temp/dataio )
add_subdirectory( datahandlers temp/datahandlers )
add_subdirectory( algorithms temp/algorithms )
add_subdirectory( pyinterface temp/pyinterface )
add_subdirectory( eval_metrics temp/eval_metrics )

if( APPLE )
   set( CMAKE_MACOSX_RPATH 0 )
   execute_process( COMMAND which python${CMAKE_PYTHON_VERSION} OUTPUT_VARIABLE PYTHON_EXE )
   string( STRIP ${PYTHON_EXE} PYTHON_EXE )
   if( DEFINED PYTHON_EXE )
      execute_process( COMMAND otool -L ${PYTHON_EXE} OUTPUT_VARIABLE OUTPUT )
      string( REPLACE "${PYTHON_EXE}:" "" OUTPUT ${OUTPUT} )
      string( STRIP ${OUTPUT} OUTPUT )
      string( REGEX MATCH ".*Python" MAC_PYTHON_LINKED_LIBRARY ${OUTPUT} )
   else( DEFINED PYTHON_EXE )
      error( "python${CMAKE_PYTHON_VERSION} not found!" )
   endif( DEFINED PYTHON_EXE )
endif( APPLE )

add_library( pyreclab SHARED
                      $<TARGET_OBJECTS:dataio>
                      $<TARGET_OBJECTS:datahandlers>
                      $<TARGET_OBJECTS:algorithms>
                      $<TARGET_OBJECTS:eval_metrics>
                      $<TARGET_OBJECTS:pyinterface> )

if( APPLE AND DEFINED MAC_PYTHON_LINKED_LIBRARY )
   target_link_libraries( pyreclab ${MAC_PYTHON_LINKED_LIBRARY} )
endif( APPLE AND DEFINED MAC_PYTHON_LINKED_LIBRARY )

set_target_properties( pyreclab PROPERTIES LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/pyreclab"
                                           SUFFIX .so
                                           SOVERSION "${MAJOR_VERSION}"
                                           VERSION "${PACKAGE_VERSION}" )

if( PYTHON )
   set( SETUP_PY_IN "${CMAKE_CURRENT_SOURCE_DIR}/pypackage/setup.py.in" )
   set( SETUP_PY    "${CMAKE_CURRENT_BINARY_DIR}/pypackage/setup.py" )
   set( OUTPUT      "${CMAKE_CURRENT_BINARY_DIR}/build/timestamp" )
   set( DEPS        "pyreclab" )

   configure_file( ${SETUP_PY_IN} ${SETUP_PY} )

   add_custom_command( OUTPUT ${OUTPUT}
                       COMMAND ${CMAKE_COMMAND} -E copy "pypackage/__init__.py" "${PROJECT_BINARY_DIR}/pyreclab/"
                       COMMAND ${PYTHON} ${SETUP_PY} bdist_egg
                       COMMAND ${CMAKE_COMMAND} -E touch ${OUTPUT}
                       DEPENDS ${DEPS} )

   add_custom_target( target ALL DEPENDS ${OUTPUT} )

   install( CODE "execute_process(COMMAND ${PYTHON} pypackage/setup.py install)" )
endif( PYTHON )


