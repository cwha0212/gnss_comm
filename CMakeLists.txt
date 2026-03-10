cmake_minimum_required(VERSION 3.8)
project(gnss_comm)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

set(CMAKE_CXX_STANDARD 14)

# Dependencies
find_package(ament_cmake REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(std_msgs REQUIRED)

# Message generation
set(msg_files
  "msg/GnssTimeMsg.msg"
  "msg/GnssEphemMsg.msg"
  "msg/GnssGloEphemMsg.msg"
  "msg/GnssMeasMsg.msg"
  "msg/GnssObsMsg.msg"
  "msg/GnssBestXYZMsg.msg"
  "msg/GnssPVTSolnMsg.msg"
  "msg/GnssSvsMsg.msg"
  "msg/GnssTimePulseInfoMsg.msg"
  "msg/StampedFloat64Array.msg"
)
rosidl_generate_interfaces(${PROJECT_NAME}
  ${msg_files}
  DEPENDENCIES std_msgs
)

# Library
set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)
find_package(Eigen3 REQUIRED)
find_package(Glog REQUIRED)

include_directories(
  include
  ${CMAKE_CURRENT_BINARY_DIR}/rosidl_generator_cpp
  ${EIGEN3_INCLUDE_DIRS}
  ${GLOG_INCLUDE_DIRS}
)

file(GLOB_RECURSE source_files ${PROJECT_SOURCE_DIR}/src/*.cpp)
add_library(gnss_comm_lib SHARED ${source_files})

target_include_directories(gnss_comm_lib PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/rosidl_generator_cpp>
  $<INSTALL_INTERFACE:include>
  ${EIGEN3_INCLUDE_DIRS}
  ${GLOG_INCLUDE_DIRS}
)

ament_target_dependencies(gnss_comm_lib
  std_msgs
  rosidl_default_runtime
)
add_dependencies(gnss_comm_lib ${PROJECT_NAME}__rosidl_typesupport_cpp)
target_link_libraries(gnss_comm_lib ${GLOG_LIBRARIES})

# Install
install(TARGETS gnss_comm_lib
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)
install(DIRECTORY include/
  DESTINATION include
)

ament_export_include_directories(include)
ament_export_libraries(gnss_comm_lib)
ament_export_dependencies(std_msgs rosidl_default_runtime)

ament_package()
