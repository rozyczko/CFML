#################################
# Commad line arguments section
#################################
cmake_minimum_required(VERSION 3.1.0 FATAL_ERROR)

#################################
# Project section
#################################
# The project is declared. It is a Fortran project.
project(Python_API Fortran)
set(LIBRARY_NAME crysfml_api)
add_library(${LIBRARY_NAME} SHARED
  Extensions/API_init.f90
  Extensions/API_Atom_TypeDef.f90
  Extensions/API_Crystallographic_Symmetry.f90
  Extensions/API_Crystal_Metrics.f90
  Extensions/API_Diffraction_Patterns.f90
  Extensions/API_Error_Messages.f90
  Extensions/API_IO_Formats.f90
  Extensions/API_Reflections_Utilities.f90
  Extensions/API_Structure_Factors.f90
  Extensions/CFML_API_Calc_Powder_Pattern.f90
  Extensions/forpy_mod.f90)

# Forpy must be preprocessed
set_source_files_properties(forpy_mod.f90 PROPERTIES COMPILE_FLAG "-cpp")

#################################
# Dependencies section
#################################
# Add a dependency to crysfml_common to keep sure that the crysfml_common library will be built first.
# See https://gitlab.kitware.com/cmake/cmake/-/issues/19226 for linking
add_dependencies(${LIBRARY_NAME} crysfml_common)

set(Python3_USE_STATIC_LIBS, OFF)
find_package(Python3 COMPONENTS Interpreter Development)

set(PYTHON_INTERPRETER_PATH ${Python3_EXECUTABLE})

if(WIN32 OR MSYS)
    # target_link_libraries(${LIBRARY_NAME} Python3::Python)
    target_include_directories(${LIBRARY_NAME} c:\\Miniconda\\envs\\test\\include)
    link_directories(c:\\Miniconda\\envs\\test\\libs c:\\Miniconda\\envs\\test\\lib)
    target_link_libraries(${LIBRARY_NAME} c:\\Miniconda\\envs\\test\\python3.dll)
endif()

# Set -Wl,-undefined,dynamic_lookup to OTHER_LDFLAGS
# Set include and linking
include_directories(${CRYSFML_COMMON_MODULE_DIRECTORY})
target_link_libraries(${LIBRARY_NAME} crysfml_common)
# Fix a problem on Mac OS X when building shared libraries
if(APPLE)
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -undefined dynamic_lookup")
endif()

# Set library extension
set_target_properties(${LIBRARY_NAME} PROPERTIES PREFIX "" OUTPUT_NAME ${LIBRARY_NAME})
set(LIBRARY_FILENAME "crysfml_api.so")
if(APPLE)
    set_target_properties(${LIBRARY_NAME} PROPERTIES SUFFIX ".so")
    set(LIBRARY_FILENAME "crysfml_api.so")
elseif(WIN32 OR MSYS)
    set_target_properties(${LIBRARY_NAME} PROPERTIES SUFFIX ".pyd")
    set(LIBRARY_FILENAME "crysfml_api.pyd")
endif()

#################################
# Documentation
#################################
find_package(Doxygen)
if (DOXYGEN_FOUND)
	set( DOXYGEN_OUTPUT_DIRECTORY doxygen )
	set( DOXYGEN_COLLABORATION_GRAPH YES )
	set( DOXYGEN_EXTRACT_ALL YES )
	set( DOXYGEN_CLASS_DIAGRAMS YES )
	set( DOXYGEN_HIDE_UNDOC_RELATIONS NO )
	set( DOXYGEN_HAVE_DOT YES )
	set( DOXYGEN_CLASS_GRAPH YES )
	set( DOXYGEN_CALL_GRAPH YES )
	set( DOXYGEN_CALLER_GRAPH YES )
	set( DOXYGEN_COLLABORATION_GRAPH YES )
	set( DOXYGEN_BUILTIN_STL_SUPPORT YES )
	set( DOXYGEN_EXTRACT_PRIVATE YES )
	set( DOXYGEN_EXTRACT_PACKAGE YES )
	set( DOXYGEN_EXTRACT_STATIC YES )
	set( DOXYGEN_EXTRACT_LOCALMETHODS YES )
	set( DOXYGEN_UML_LOOK YES )
	set( DOXYGEN_UML_LIMIT_NUM_FIELDS 50 )
	set( DOXYGEN_TEMPLATE_RELATIONS YES )
	set( DOXYGEN_DOT_GRAPH_MAX_NODES 100 )
	set( DOXYGEN_MAX_DOT_GRAPH_DEPTH 0 )
	set( DOXYGEN_DOT_TRANSPARENT YES )
	set( DOXYGEN_EXCLUDE_PATTERNS *.f90 )
	doxygen_add_docs( doxygen ${CMAKE_CURRENT_SOURCE_DIR} )
else (DOXYGEN_FOUND)
  message("Doxygen is not installed - no documentation will be generated")
endif (DOXYGEN_FOUND)


#################################
# Tests
#################################
# Add tests
add_test(NAME "Python_API_Tests" COMMAND ${PYTHON_INTERPRETER_PATH} -m unittest discover ${CMAKE_CURRENT_BINARY_DIR}/../Tests)
add_custom_target(Python_Files ALL
        COMMAND
        ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_CURRENT_SOURCE_DIR}/../Tests
        ${CMAKE_CURRENT_BINARY_DIR}/../Tests

        COMMAND
        ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_CURRENT_BINARY_DIR}/../Tests/CFML_api
        )

add_dependencies(Python_Files ${LIBRARY_NAME})

if(NOT PYSETUP)
    add_custom_target(Python_SO_Files ALL
        COMMAND
        ${CMAKE_COMMAND} -E copy
        ${CMAKE_CURRENT_BINARY_DIR}/${LIBRARY_FILENAME}
        ${CMAKE_CURRENT_BINARY_DIR}/../Tests/CFML_api
        )
    add_dependencies(Python_SO_Files ${LIBRARY_NAME})
endif()

#################################
# Install section
#################################
# The rules for installing the library.
install(DIRECTORY "" DESTINATION ${PYTHON_API_PREFIX}/CFML_api FILES_MATCHING PATTERN "*.py" PATTERN "Extensions" EXCLUDE)
if(WIN32 OR MSYS)
install(TARGETS ${LIBRARY_NAME} RUNTIME DESTINATION ${PYTHON_API_PREFIX}/CFML_api)
else()
install(TARGETS ${LIBRARY_NAME} LIBRARY DESTINATION ${PYTHON_API_PREFIX}/CFML_api)
endif()

# Installing example files
install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../Examples" DESTINATION ${PYTHON_API_PREFIX} FILES_MATCHING PATTERN "*.py")
install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../Examples/Data" DESTINATION ${PYTHON_API_PREFIX}/Examples FILES_MATCHING PATTERN "*.cfl" PATTERN "*.cif")
#
# # Installing tests files
install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../Tests" DESTINATION ${PYTHON_API_PREFIX} FILES_MATCHING PATTERN "*.py")
#
# # Installing documentation files
install(DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/doxygen/html" DESTINATION ${PYTHON_API_PREFIX}/Doc OPTIONAL)