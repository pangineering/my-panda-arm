cmake_minimum_required(VERSION 3.0.2)
project(robot_control)

## Compile as C++11, supported in ROS Kinetic and newer
# add_compile_options(-std=c++11)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  controller_interface
  controller_manager
  geometry_msgs
  message_generation
  pluginlib
  realtime_tools
  robot_msgs
  franka_hw
  roscpp
  sensor_msgs
  std_srvs
  tf
  tf2_msgs
)

find_package(Franka 0.9.0 QUIET)
if(NOT Franka_FOUND)
  find_package(Franka 0.8.0 REQUIRED)
endif()

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES franka_state_controller
  CATKIN_DEPENDS
    controller_interface
    controller_manager
    robot_msgs
    franka_hw
    geometry_msgs
    pluginlib
    realtime_tools
    roscpp
    sensor_msgs
    tf2_msgs
    std_srvs
  DEPENDS Franka
)


## franka_state_controller
add_library(franka_state_controller
  src/franka_state_controller.cpp
)
if (Franka_VERSION GREATER_EQUAL 0.9)
    target_compile_definitions(franka_state_controller PUBLIC ENABLE_BASE_ACCELERATION)
endif()
add_dependencies(franka_state_controller
  ${${PROJECT_NAME}_EXPORTED_TARGETS}
  ${catkin_EXPORTED_TARGETS}
)

target_link_libraries(franka_state_controller
  ${Franka_LIBRARIES}
  ${catkin_LIBRARIES}
)

target_include_directories(franka_state_controller SYSTEM PUBLIC
  ${Franka_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
)
target_include_directories(franka_state_controller PUBLIC
  include
)

## franka_control_node
add_executable(franka_control_node
  src/franka_control_node.cpp
)

add_dependencies(franka_control_node
  ${${PROJECT_NAME}_EXPORTED_TARGETS}
  ${catkin_EXPORTED_TARGETS}
)

target_link_libraries(franka_control_node
  ${Franka_LIBRARIES}
  ${catkin_LIBRARIES}
)

target_include_directories(franka_control_node SYSTEM PUBLIC
  ${Franka_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
)

add_executable(franka_combined_control_node
    src/franka_combined_control_node.cpp
)

add_dependencies(franka_combined_control_node
    ${catkin_EXPORTED_TARGETS}
)

target_link_libraries(franka_combined_control_node
  ${catkin_LIBRARIES}
)

target_include_directories(franka_combined_control_node SYSTEM PUBLIC
  ${catkin_INCLUDE_DIRS}
)

## Installation
install(TARGETS franka_state_controller
                franka_control_node
                franka_combined_control_node
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
install(FILES franka_controller_plugins.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)