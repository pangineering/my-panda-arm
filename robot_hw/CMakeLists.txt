cmake_minimum_required(VERSION 3.0.2)
project(robot_hw)

## Compile as C++11, supported in ROS Kinetic and newer
# add_compile_options(-std=c++11)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  actionlib
  combined_robot_hw
  controller_interface
  hardware_interface
  joint_limits_interface
  message_generation
  pluginlib
  robot_msgs
  roscpp
  std_srvs
  urdf
)

find_package(Franka 0.9.0 QUIET)
if(NOT Franka_FOUND)
  find_package(Franka 0.8.0 REQUIRED)
endif()


add_library(robot_hw
  src/control_mode.cpp
  src/franka_hw.cpp
  src/franka_combinable_hw.cpp
  src/franka_combined_hw.cpp
  src/resource_helpers.cpp
  src/trigger_rate.cpp
)

add_dependencies(robot_hw
  ${${PROJECT_NAME}_EXPORTED_TARGETS}
  ${catkin_EXPORTED_TARGETS}
)

target_link_libraries(robot_hw
  ${Franka_LIBRARIES}
  ${catkin_LIBRARIES}
  franka_control_services
)

target_include_directories(robot_hw SYSTEM PUBLIC
  ${Franka_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
)
target_include_directories(robot_hw PUBLIC
  include
)


## franka_control_services
add_library(franka_control_services
  src/services.cpp
)

add_dependencies(franka_control_services
  ${${PROJECT_NAME}_EXPORTED_TARGETS}
  ${catkin_EXPORTED_TARGETS}
)

target_link_libraries(franka_control_services
  ${Franka_LIBRARIES}
  ${catkin_LIBRARIES}
)

target_include_directories(franka_control_services SYSTEM PUBLIC
  ${Franka_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
)

target_include_directories(franka_control_services PUBLIC
  include
)

## Installation
install(TARGETS robot_hw franka_control_services
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
# install(DIRECTORY include/${PROJECT_NAME}/
#   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
# )

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
  add_format_target(robot_hw FILES ${SOURCES} ${HEADERS})
  add_tidy_target(robot_hw
    FILES ${SOURCES}
    DEPENDS robot_hw
  )
endif()
