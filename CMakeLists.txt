cmake_minimum_required(VERSION 2.8.3)
project(dlib_facedetector)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages

SET(CMAKE_CXX_FLAGS "-Wall  -O2 -g -march=native" )

#use PkgConfig
find_package(PkgConfig REQUIRED)

find_package(dlib 18.18.1 REQUIRED)

# This driver uses Boost threads
find_package(Boost REQUIRED COMPONENTS thread)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  image_transport
  sensor_msgs
  people_msgs
  camera_info_manager
  roslint
  nodelet
  dynamic_reconfigure
)

find_package(Boost REQUIRED COMPONENTS system thread)

## System dependencies are found with CMake's conventions
# find_package(Boost REQUIRED COMPONENTS system)


## Uncomment this if the package has a setup.py. This macro ensures
## modules and global scripts declared therein get installed
## See http://ros.org/doc/api/catkin/html/user_guide/setup_dot_py.html
# catkin_python_setup()

################################################
## Declare ROS messages, services and actions ##
################################################

## To declare and build messages, services or actions from within this
## package, follow these steps:
## * Let MSG_DEP_SET be the set of packages whose message types you use in
##   your messages/services/actions (e.g. std_msgs, actionlib_msgs, ...).
## * In the file package.xml:
##   * add a build_depend and a run_depend tag for each package in MSG_DEP_SET
##   * If MSG_DEP_SET isn't empty the following dependencies might have been
##     pulled in transitively but can be declared for certainty nonetheless:
##     * add a build_depend tag for "message_generation"
##     * add a run_depend tag for "message_runtime"
## * In this file (CMakeLists.txt):
##   * add "message_generation" and every package in MSG_DEP_SET to
##     find_package(catkin REQUIRED COMPONENTS ...)
##   * add "message_runtime" and every package in MSG_DEP_SET to
##     catkin_package(CATKIN_DEPENDS ...)
##   * uncomment the add_*_files sections below as needed
##     and list every .msg/.srv/.action file to be processed
##   * uncomment the generate_messages entry below
##   * add every package in MSG_DEP_SET to generate_messages(DEPENDENCIES ...)

## Generate messages in the 'msg' folder
# add_message_files(
#   FILES
#   Message1.msg
#   Message2.msg
# )

## Generate services in the 'srv' folder
# add_service_files(
#   FILES
#   Service1.srv
#   Service2.srv
# )

## Generate actions in the 'action' folder
# add_action_files(
#   FILES
#   Action1.action
#   Action2.action
# )

## Generate added messages and services with any dependencies listed here
#generate_messages(
#  DEPENDENCIES
#  std_msgs  # Or other packages containing msgs
#)

#add dynamic reconfigure api
generate_dynamic_reconfigure_options(
  cfg/dlib_facedetector.cfg
)


MESSAGE(INFO ${dlib_INSTALL_PATH})
MESSAGE(INFO ${dlib_INCLUDE_DIRS})

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if you package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
  INCLUDE_DIRS include
  LIBRARIES dlib_facedetector_nodelet
  CATKIN_DEPENDS roscpp nodelet
  DEPENDS system_lib TinyXML
  DEPENDS boost
)

link_directories(${dlib_LIBRARY_DIRS})

###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
# include_directories(include)
include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${dlib_INCLUDE_DIRS}
)

## nodelet library
add_library(dlib_facedetector_nodelet src/nodelet.cpp )

# make sure configure headers are built before any node using them
add_dependencies(dlib_facedetector_nodelet ${PROJECT_NAME}_gencfg)

## Specify libraries to link a library or executable target against
target_link_libraries(dlib_facedetector_nodelet
  ${catkin_LIBRARIES}
  ${dlib_LIBRARIES}
  ${Boost_LIBRARIES}
  yaml-cpp
)

set_target_properties(dlib_facedetector_nodelet PROPERTIES COMPILE_FLAGS "-mavx")

#############
## style guide ##
#############
set(ENABLE_CPPLINT 1)
include(${CMAKE_CURRENT_SOURCE_DIR}/stylecheck/CpplintWrapper.cmake)
CPPLINT_RECURSIVE(cpplint_include
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${CMAKE_CURRENT_BINARY_DIR}/include)
CPPLINT_RECURSIVE(cpplint_src
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${CMAKE_CURRENT_BINARY_DIR}/src)
add_dependencies(dlib_facedetector_nodelet cpplint_src cpplint_include)

#############
## Install ##
#############

# all install targets should use catkin DESTINATION variables
# See http://ros.org/doc/api/catkin/html/adv_user_guide/variables.html

## Mark executable scripts (Python etc.) for installation
## in contrast to setup.py, you can choose the destination
# install(PROGRAMS
#   scripts/my_python_script
#   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark executables and/or libraries for installation
install(TARGETS dlib_facedetector_nodelet
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

## Mark cpp header files for installation
# install(DIRECTORY include/${PROJECT_NAME}/
#   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
#   FILES_MATCHING PATTERN "*.h"
#   PATTERN ".svn" EXCLUDE
# )

## Mark other files for installation (e.g. launch and bag files, etc.)
install(DIRECTORY launch
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  FILES_MATCHING PATTERN "*.launch"
)

install(FILES nodelet.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

#############
## Testing ##
#############

## Add gtest based cpp test target and link libraries
# catkin_add_gtest(${PROJECT_NAME}-test test/test_ximea2.cpp)
# if(TARGET ${PROJECT_NAME}-test)
#   target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME})
# endif()

## Add folders to be run by python nosetests
# catkin_add_nosetests(test)

###########
##roslint##
###########
set(ROSLINT_CPP_OPTS  "--filter=-whitespace/line_length,-readability/todo" )
roslint_cpp()


#workaround for qtcreator ide integration. do not remove!
file(GLOB_RECURSE NODE_DUMMY_TARGETS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.h *.cfg *.yaml *.xml *.launch)
add_custom_target(node_dummy_target SOURCES ${NODE_DUMMY_TARGETS})
