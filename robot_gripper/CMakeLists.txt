cmake_minimum_required(VERSION 3.0.2)
project(robot_gripper)

## Compile as C++11, supported in ROS Kinetic and newer
# add_compile_options(-std=c++11)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  actionlib
  actionlib_msgs
  control_msgs
  message_generation
  roscpp
  sensor_msgs
  xmlrpcpp
)


find_package(Franka 0.9.0 QUIET)
if(NOT Franka_FOUND)
  find_package(Franka 0.8.0 REQUIRED)
endif()

add_action_files(
  DIRECTORY action
  FILES Grasp.action
        Homing.action
        Stop.action
        Move.action
)

add_message_files(
  DIRECTORY msg
  FILES GraspEpsilon.msg
)

generate_messages(DEPENDENCIES actionlib_msgs)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES robot_gripper
  CATKIN_DEPENDS roscpp
                 message_runtime
                 control_msgs
                 actionlib
                 sensor_msgs
                 xmlrpcpp
                 actionlib_msgs
  DEPENDS Franka
)

add_library(robot_gripper
  src/franka_gripper.cpp
)

add_dependencies(robot_gripper
  ${${PROJECT_NAME}_EXPORTED_TARGETS}
  ${catkin_EXPORTED_TARGETS}
  ${PROJECT_NAME}_generate_messages_cpp
)

target_link_libraries(robot_gripper
  ${Franka_LIBRARIES}
  ${catkin_LIBRARIES}
)

target_include_directories(robot_gripper SYSTEM PUBLIC
  ${Franka_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
)
target_include_directories(robot_gripper PUBLIC
  include
)

add_executable(robot_gripper_node
  src/franka_gripper_node.cpp
)

add_dependencies(robot_gripper_node
  ${${PROJECT_NAME}_EXPORTED_TARGETS}
  ${catkin_EXPORTED_TARGETS}
  robot_gripper
)

target_link_libraries(robot_gripper_node
   ${catkin_LIBRARIES}
   robot_gripper
)

target_include_directories(robot_gripper_node SYSTEM PUBLIC
  ${catkin_INCLUDE_DIRS}
)

## Installation
install(TARGETS robot_gripper
                robot_gripper_node
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)
install(DIRECTORY launch
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
install(DIRECTORY config
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

## Tools
include(${CMAKE_CURRENT_LIST_DIR}/../cmake/ClangTools.cmake OPTIONAL
  RESULT_VARIABLE CLANG_TOOLS
)
if(CLANG_TOOLS)
  file(GLOB_RECURSE SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
  file(GLOB_RECURSE HEADERS
    ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.h
  )
  add_format_target(robot_gripper FILES ${SOURCES} ${HEADERS})
  add_tidy_target(robot_gripper
    FILES ${SOURCES}
    DEPENDS robot_gripper robot_gripper_node
  )
endif()
