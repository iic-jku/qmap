cmake_minimum_required(VERSION 3.14...3.19)

project(qmap
        LANGUAGES CXX
        VERSION 1.3.2
        DESCRIPTION "QMAP  - A JKQ library for mapping of quantum circuits to quantum architectures"
        )

# enable organization of targets into folders
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# configuration options
option(DEPLOY "Configure for deployment")
option(BINDINGS "Configure for building Python bindings")
option(COVERAGE "Configure for coverage report generation")
option(GENERATE_POSITION_INDEPENDENT_CODE "Generate position independent code")
option(BUILD_QMAP_TESTS "Also build tests for QMAP project")

# build type settings
set(default_build_type "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
	message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
	set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE STRING "Choose the type of build." FORCE)
	set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

macro(check_submodule_present MODULENAME)
	if(NOT EXISTS "${PROJECT_SOURCE_DIR}/extern/${MODULENAME}/CMakeLists.txt")
		message(FATAL_ERROR "${MODULENAME} submodule not cloned properly. Please run `git submodule update --init --recursive` from the main project directory")
	endif()
endmacro()

check_submodule_present(qfr)

# Add path for custom modules
list (APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# search for Z3
find_package(Z3)
if(Z3_FOUND AND NOT TARGET z3::z3lib)
	message(STATUS "Found Z3 with version ${Z3_VERSION_STRING}")
	add_library(z3::z3lib IMPORTED INTERFACE)
	set_property(TARGET z3::z3lib PROPERTY INTERFACE_INCLUDE_DIRECTORIES ${Z3_CXX_INCLUDE_DIRS})
	set_property(TARGET z3::z3lib PROPERTY INTERFACE_LINK_LIBRARIES ${Z3_LIBRARIES})
else()
	message(WARNING "Did not find Z3. Exact library and other depending target will not be available")
endif()

# add main library code
add_subdirectory(src)

# add test code
if (BUILD_QMAP_TESTS)
	enable_testing()
	include(GoogleTest)
	add_subdirectory(test)
endif ()

# add apps if this is the main project
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
	add_subdirectory(apps)
endif()

# add Python binding code
if(BINDINGS AND Z3_FOUND)
	add_subdirectory(jkq/qmap)
endif()
