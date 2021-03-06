cmake_minimum_required(VERSION 2.8.3)
project(force_controller)

catkin_python_setup()

find_package(catkin REQUIRED COMPONENTS
 roscpp
 geometry_msgs
 sensor_msgs
 baxter_core_msgs
 kdl_parser
 dynamic_reconfigure
 message_generation
 genmsg
)

# ROS Independent packages
find_package(cmake_modules REQUIRED)
find_package(orocos_kdl)
find_package(Eigen REQUIRED)

## Dynamic Reconfigure: takes path to all cfg files that we want to build.
generate_dynamic_reconfigure_options(
  cfg/force_error_constants.cfg
  )

# Generate message files in the 'msg' folder
add_message_files(
  DIRECTORY 
  msg 
  FILES 
  setPoint.msg)

# Generate services in the 'srv' folder
add_service_files(
  DIRECTORY 
  srv 
  FILES 
  forceControl.srv)

## Generate added messages and services with any dependencies listed here
generate_messages(DEPENDENCIES
   baxter_core_msgs
   geometry_msgs
   sensor_msgs
   std_msgs
)

###################################
## catkin specific configuration ##
###################################
catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}

  CATKIN_DEPENDS
	baxter_core_msgs
	sensor_msgs
	geometry_msgs
	dynamic_reconfigure
	message_runtime
)

include_directories( 
  include
  ${catkin_INCLUDE_DIRS} 
  ${EIGEN_INCLUDE_DIRS} 
  ${dynamic_reconfigure_PACKAGE_PATH}/cmake/cfgbuild.cmake} 
)

# C++0x support - not quite the same as final C++11!
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")

# Kinematics Library
add_library(mykinematics src/kinematics.cpp)
target_link_libraries(mykinematics ${catkin_LIBRARIES})

# # 1a. Force Controller Service Server 
# add_executable       (force_cont_service src/force_controller_service.cpp)
# target_link_libraries(force_cont_service ${catkin_LIBRARIES})
# add_dependencies     (force_cont_service %{PROJECT_NAME}_gencfg) # Builds dyn_rec headers before use.
# add_dependencies     (force_cont_service ${catkin_EXPORTED_TARGETS}) # setPoint.msg??
# add_dependencies     (mykinematics ${catkin_EXPORTED_TARGETS})
# target_link_libraries(force_cont_service ${catkin_LIBRARIES} mykinematics) # links kin lib.

# # 1b. Force Controller Service Client 
#  add_executable       (force_control_srv_client  src/force_control_srv_client/force_control_srv_client.cpp)
#  target_link_libraries(force_control_srv_client ${catkin_LIBRARIES})
#  add_dependencies     (force_control_srv_client %{PROJECT_NAME}_generate_messages_cpp)
#  add_dependencies     (force_control_srv_client ${catkin_EXPORTED_TARGETS}) # setPoint.msg??

# 2a. Force Controller (Set Point Subscriber)
add_executable       (force_cont_topic src/force_controller_topic.cpp)
target_link_libraries(force_cont_topic ${catkin_LIBRARIES})
add_dependencies     (mykinematics ${catkin_EXPORTED_TARGETS})
add_dependencies     (force_cont_topic %{PROJECT_NAME}_gencfg)    # Builds dyn_rec headers before use.
add_dependencies     (force_cont_topic ${catkin_EXPORTED_TARGETS}) # setPoint.msg??
target_link_libraries(force_cont_topic ${catkin_LIBRARIES} mykinematics) # lins kin lib

# 2b. Force Controller Set Point Publisher
add_executable       (setPoint_pub src/setPoint/setPoint_publisher.cpp)
target_link_libraries(setPoint_pub ${catkin_LIBRARIES})
add_dependencies     (setPoint_pub %{PROJECT_NAME}_generate_message_cpp)
add_dependencies     (setPoint_pub ${catkin_EXPORTED_TARGETS}) # setPoint.msg??

# Install
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})
