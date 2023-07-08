cmake_minimum_required(VERSION 3.0.2)
project(robot_msgs)

## Compile as C++11, supported in ROS Kinetic and newer
# add_compile_options(-std=c++11)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  actionlib_msgs
  message_generation
  std_msgs
)



add_message_files(FILES Errors.msg FrankaState.msg)

add_service_files(FILES
  SetCartesianImpedance.srv
  SetEEFrame.srv
  SetForceTorqueCollisionBehavior.srv
  SetFullCollisionBehavior.srv
  SetJointImpedance.srv
  SetKFrame.srv
  SetLoad.srv
)

add_action_files(FILES ErrorRecovery.action)

generate_messages(DEPENDENCIES std_msgs actionlib_msgs)

catkin_package(CATKIN_DEPENDS message_runtime std_msgs actionlib_msgs)


