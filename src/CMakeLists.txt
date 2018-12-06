FILE( GLOB_RECURSE KDL_SRCS [^.]*.cpp [^.]*.cxx)
FILE( GLOB KDL_HPPS [^.]*.hpp [^.]*.inl)
FILE( GLOB UTIL_HPPS utilities/[^.]*.h utilities/[^.]*.hpp)

if(HAVE_STL_CONTAINER_INCOMPLETE_TYPES)
    SET(KDL_USE_NEW_TREE_INTERFACE_DEFAULT Off)
ELSE(HAVE_STL_CONTAINER_INCOMPLETE_TYPES)
    SET(KDL_USE_NEW_TREE_INTERFACE_DEFAULT On)
ENDIF(HAVE_STL_CONTAINER_INCOMPLETE_TYPES)

SET(KDL_USE_NEW_TREE_INTERFACE ${KDL_USE_NEW_TREE_INTERFACE_DEFAULT} CACHE BOOL "Use the new KDL Tree interface")

#Sanity check, inform the user
IF(NOT HAVE_STL_CONTAINER_INCOMPLETE_TYPES AND NOT KDL_USE_NEW_TREE_INTERFACE)
    MESSAGE(WARNING "You have chosen to use the current Tree Interface, but your platform doesn't support containers of "
        "incomplete types, this configuration is likely invalid")
ENDIF()
SET(LIB_TYPE STATIC)

#### Settings for rpath
IF(${CMAKE_MINIMUM_REQUIRED_VERSION} VERSION_GREATER "2.8.12")
    MESSAGE(AUTHOR_WARNING "CMAKE_MINIMUM_REQUIRED_VERSION is now ${CMAKE_MINIMUM_REQUIRED_VERSION}. This check can be removed.")
ENDIF()
IF(NOT (CMAKE_VERSION VERSION_LESS 2.8.12))
    IF(NOT MSVC)
        #add the option to disable RPATH
        OPTION(OROCOSKDL_ENABLE_RPATH "Enable RPATH during installation" TRUE)
        MARK_AS_ADVANCED(OROCOSKDL_ENABLE_RPATH)
    ENDIF(NOT MSVC)

    IF(OROCOSKDL_ENABLE_RPATH)
        #Configure RPATH
        SET(CMAKE_MACOSX_RPATH TRUE) #enable RPATH on OSX. This also suppress warnings on CMake >= 3.0
        # when building, don't use the install RPATH already
        # (but later on when installing)
        SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
        #build directory by default is built with RPATH
        SET(CMAKE_SKIP_BUILD_RPATH  FALSE)

        #This is relative RPATH for libraries built in the same project
        #I assume that the directory is
        # - install_dir/something for binaries
        # - install_dir/lib for libraries
        LIST(FIND CMAKE_PLATFORM_IMPLICIT_LINK_DIRECTORIES "${CMAKE_INSTALL_PREFIX}/lib" isSystemDir)
        IF("${isSystemDir}" STREQUAL "-1")
            FILE(RELATIVE_PATH _rel_path "${CMAKE_INSTALL_PREFIX}/bin" "${CMAKE_INSTALL_PREFIX}/lib")
            IF (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
                SET(CMAKE_INSTALL_RPATH "@loader_path/${_rel_path}")
            ELSE()
                SET(CMAKE_INSTALL_RPATH "\$ORIGIN/${_rel_path}")
            ENDIF()
        ENDIF("${isSystemDir}" STREQUAL "-1")
        # add the automatically determined parts of the RPATH
        # which point to directories outside the build tree to the install RPATH
        SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE) #very important!
    ENDIF()
ENDIF()
#####end RPATH

ADD_LIBRARY(orocos-kdl ${LIB_TYPE} ${KDL_SRCS})
