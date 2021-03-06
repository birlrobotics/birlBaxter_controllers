cmake_minimum_required(VERSION 2.8.3)
project(imped_control)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  controller_interface
  control_toolbox
  baxter_core_msgs
  pluginlib
  pr2_controller_interface
  pr2_mechanism_model
  roscpp
)

## System dependencies are found with CMake's conventions
find_package(Boost REQUIRED COMPONENTS system)
find_package(PkgConfig REQUIRED)
pkg_check_modules(YAML_CPP REQUIRED yaml-cpp)

find_path(YAML_CPP_INCLUDE_DIR
          NAMES yaml_cpp.h
          PATHS ${YAML_CPP_INCLUDE_DIRS})

find_library(YAML_CPP_LIBRARY
             NAMES YAML_CPP
             PATHS ${YAML_CPP_LIBRARY_DIRS})

if(${YAML_CPP_VERSION} VERSION_LESS 0.5)
  add_definitions(-DDEPRECATED_YAML_CPP_VERSION)
endif()

## Uncomment this if the package has a setup.py. This macro ensures
## modules and global scripts declared therein get installed
## See http://ros.org/doc/api/catkin/html/user_guide/setup_dot_py.html
# catkin_python_setup()

catkin_package(
  CATKIN_DEPENDS
  pluginlib
  pr2_controller_interface
  pr2_mechanism_model  
  controller_interface
  control_toolbox
  baxter_core_msgs
  roscpp
  DEPENDS yaml-cpp
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIR}
)

add_library(${PROJECT_NAME}
   src/imped_control.cpp
   src/utility.cpp
)

target_link_libraries(${PROJECT_NAME}
  ${catkin_LIBRARIES}
  ${Boost_LIBRARIES}
  ${YAML_CPP_LIBRARIES}

)

add_dependencies(${PROJECT_NAME}
  ${catkin_EXPORTED_TARGETS}
  baxter_core_msgs_gencpp
)
