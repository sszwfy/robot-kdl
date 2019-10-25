INCLUDE_DIRECTORIES(${PROJ_SOURCE_DIR}/src ${PROJ_SOURCE_DIR}/models ${PROJ_BINARY_DIR}/src)

add_executable(chainiksolverpos_lma_demo chainiksolverpos_lma_demo.cpp puma560.cpp)
TARGET_LINK_LIBRARIES(chainiksolverpos_lma_demo orocos-kdl)


