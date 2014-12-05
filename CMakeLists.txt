cmake_minimum_required(VERSION 2.4.6)
include($ENV{ROS_ROOT}/core/rosbuild/rosbuild.cmake)

rosbuild_init()

set(BUILD_DIR ${CMAKE_CURRENT_SOURCE_DIR}/build)
set(TARBALL_NAME lapackpp-2.5.4.tar.gz)
set(TARBALL_URL http://downloads.sourceforge.net/project/lapackpp/${TARBALL_NAME})
set(EXTRACTED_DIR ${BUILD_DIR}/lapackpp-2.5.4)

add_custom_command(
    OUTPUT ${BUILD_DIR}/lapackpp-2.5.4.tar.gz
    mkdir -p ${BUILD_DIR}
    COMMAND wget -q -P ${BUILD_DIR} ${TARBALL_URL}
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)

add_custom_command(
    OUTPUT ${EXTRACTED_DIR}/configure
    COMMAND tar xzf ${TARBALL_NAME}
    DEPENDS ${BUILD_DIR}/lapackpp-2.5.4.tar.gz
    WORKING_DIRECTORY ${BUILD_DIR}
)

add_custom_command(
  OUTPUT ${EXTRACTED_DIR}/src/.libs/liblapackpp.so
  COMMAND ${EXTRACTED_DIR}/configure 
  COMMAND unset MAKEFLAGS && unset MAKELEVEL && unset MFLAGS && make LDFLAGS=-lgcc
  COMMAND mkdir -p ${CMAKE_CURRENT_SOURCE_DIR}/include
  COMMAND mkdir -p ${CMAKE_CURRENT_SOURCE_DIR}/lib
  COMMAND cp -r ${EXTRACTED_DIR}/include/* ${CMAKE_CURRENT_SOURCE_DIR}/include
  COMMAND cp ${EXTRACTED_DIR}/src/.libs/* ${CMAKE_CURRENT_SOURCE_DIR}/lib
  DEPENDS ${EXTRACTED_DIR}/configure
  WORKING_DIRECTORY ${EXTRACTED_DIR}
)

add_library(${PROJECT_NAME} ${EXTRACTED_DIR}/src/.libs/liblapackpp.so)
set_target_properties(${PROJECT_NAME} PROPERTIES LINKER_LANGUAGE CXX)
add_dependencies(${PROJECT_NAME} ${EXTRACTED_DIR}/src/.libs/liblapackpp.so)

set_directory_properties(PROPERTIES ADDITIONAL_MAKE_CLEAN_FILES "${EXTRACTED_DIR}")
