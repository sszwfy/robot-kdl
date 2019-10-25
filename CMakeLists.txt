#
# Test CMake version
#
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
#MARK_AS_ADVANCED( FORCE CMAKE_BACKWARDS_COMPATIBILITY )


###################################################
#                                                 #
#       Start project customization section       #
#                                                 #
###################################################

PROJECT(orocos_kdl)

SET( PROJ_SOURCE_DIR ${orocos_kdl_SOURCE_DIR} )
SET( PROJ_BINARY_DIR ${orocos_kdl_BINARY_DIR} )


IF(NOT CMAKE_INSTALL_PREFIX)
  SET( CMAKE_INSTALL_PREFIX /usr/local/ CACHE PATH "Installation directory" FORCE)
  MESSAGE( STATUS "Setting installation directory to ${CMAKE_INSTALL_PREFIX}" )
ENDIF(NOT CMAKE_INSTALL_PREFIX)

IF ( NOT CMAKE_BUILD_TYPE )
  SET(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build, options are: None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug Release RelWithDebInfo MinSizeRel." FORCE)
  MESSAGE( STATUS "Setting build type to '${CMAKE_BUILD_TYPE}'" )
ELSE ( NOT CMAKE_BUILD_TYPE )
  MESSAGE( STATUS "Build type set to '${CMAKE_BUILD_TYPE}' by user." )
ENDIF ( NOT CMAKE_BUILD_TYPE )

SET( KDL_CFLAGS "")

find_package(Eigen 3 QUIET)
if(NOT Eigen_FOUND)
  include(${PROJ_SOURCE_DIR}/config/FindEigen3.cmake)
  include_directories(${EIGEN3_INCLUDE_DIR})
endif()

# Check the platform STL containers capabilities
include(config/CheckSTLContainers.cmake)
CHECK_STL_CONTAINERS()

# Set the default option appropriately
if(HAVE_STL_CONTAINER_INCOMPLETE_TYPES)
    set(KDL_USE_NEW_TREE_INTERFACE_DEFAULT Off)
else(HAVE_STL_CONTAINER_INCOMPLETE_TYPES)
    set(KDL_USE_NEW_TREE_INTERFACE_DEFAULT On)
endif(HAVE_STL_CONTAINER_INCOMPLETE_TYPES)

# Allow the user to select the Tree API version to use
set(KDL_USE_NEW_TREE_INTERFACE ${KDL_USE_NEW_TREE_INTERFACE_DEFAULT} CACHE BOOL "Use the new KDL Tree interface")

# The new interface requires the use of shared pointers
if(KDL_USE_NEW_TREE_INTERFACE)
    # We need shared_ptr from boost since not all compilers are c++11 capable
    find_package(Boost REQUIRED)
    include_directories(${Boost_INCLUDE_DIRS})
endif(KDL_USE_NEW_TREE_INTERFACE)
INCLUDE (${PROJ_SOURCE_DIR}/config/DependentOption.cmake)

OPTION(ENABLE_EXAMPLES ON "Enable building of examples")
ADD_SUBDIRECTORY( src )
ADD_SUBDIRECTORY( examples )