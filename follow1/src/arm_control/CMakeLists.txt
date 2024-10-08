cmake_minimum_required(VERSION 3.0.2)
project(arm_control)

## Compile as C++11, supported in ROS Kinetic and newer
add_compile_options(-std=c++11)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CURSES_NEED_NCURSES TRUE)
find_package(catkin REQUIRED COMPONENTS
  dynamic_reconfigure
  geometry_msgs
  roscpp
  rospy
  std_msgs
  tf
  message_generation
  message_runtime
  kdl_parser
)

add_message_files(
  FILES
  PosCmd.msg
  JointControl.msg 
  JointInformation.msg 
  ChassisCtrl.msg
  MagicCmd.msg
)

generate_messages(
  DEPENDENCIES
  std_msgs
  geometry_msgs#   std_msgs
  # arx
)

generate_dynamic_reconfigure_options(
  cfg/reconfig.cfg
#   cfg/DynReconf1.cfg
#   cfg/DynReconf2.cfg
)

catkin_package(
 INCLUDE_DIRS include
 CATKIN_DEPENDS dynamic_reconfigure geometry_msgs roscpp rospy std_msgs tf #message_generation message_runtime
 CATKIN_DEPENDS kdl_parser
)

include_directories(
include
  ${catkin_INCLUDE_DIRS}
  ${roscpp_INCLUDE_DIRS}
)

add_library(arm_control
   SHARED
   src/App/arm_control.cpp

   src/Hardware/math_ops.cpp
   src/Hardware/motor.cpp
   src/Hardware/teleop.cpp
   src/utility.cpp

  #  src/Hardware/can.cpp
  #  src/App/keyboard.cpp
  #  src/App/solve.cpp
  #  src/App/play.cpp
  #  src/libcan/SocketCAN.cpp
)

add_executable(follow_1 src/arx_5v.cpp)
add_dependencies(follow_1 ${PROJECT_NAME}_gencfg)

add_executable(follow_1_pos src/arx_5v_pos.cpp)
add_dependencies(follow_1_pos ${PROJECT_NAME}_gencfg)

if(CMAKE_SYSTEM_PROCESSOR MATCHES "arm.*|arm|aarch64")
    # Link libraries for ARM architecture
    target_link_libraries(follow_1 
        ${catkin_LIBRARIES} 
        arm_control 
        ${CMAKE_CURRENT_SOURCE_DIR}/src/App/libarmkey.so 
        ${CMAKE_CURRENT_SOURCE_DIR}/src/App/libarmplay.so
        ${CMAKE_CURRENT_SOURCE_DIR}/src/App/libarmsolve.so
        ${CMAKE_CURRENT_SOURCE_DIR}/src/Hardware/libarmcana.so
        ${CMAKE_CURRENT_SOURCE_DIR}/src/libcan/libarmcan.a
        
        kdl_parser
    )
    target_link_libraries(follow_1 _pos
        ${catkin_LIBRARIES} 
        arm_control 
        ${CMAKE_CURRENT_SOURCE_DIR}/src/App/libarmkey.so 
        ${CMAKE_CURRENT_SOURCE_DIR}/src/App/libarmplay.so
        ${CMAKE_CURRENT_SOURCE_DIR}/src/App/libarmsolve.so
        ${CMAKE_CURRENT_SOURCE_DIR}/src/Hardware/libarmcana.so
        ${CMAKE_CURRENT_SOURCE_DIR}/src/libcan/libarmcan.a
        
        kdl_parser
    )
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86_64)|(AMD64)|(^i686)")
    # Link libraries for x86 architecture
    target_link_libraries(follow_1 
        ${catkin_LIBRARIES} 
        arm_control 
        ${CMAKE_CURRENT_SOURCE_DIR}/src/App/libplay.so 
        ${CMAKE_CURRENT_SOURCE_DIR}/src/App/libsolve.so 
        ${CMAKE_CURRENT_SOURCE_DIR}/src/Hardware/libcana.so 
        ${CMAKE_CURRENT_SOURCE_DIR}/src/libcan/libcan.a 
        ${CMAKE_CURRENT_SOURCE_DIR}/src/App/libkey.so
        kdl_parser
    )
    target_link_libraries(follow_1_pos
        ${catkin_LIBRARIES} 
        arm_control 
        ${CMAKE_CURRENT_SOURCE_DIR}/src/App/libplay.so 
        ${CMAKE_CURRENT_SOURCE_DIR}/src/App/libsolve.so 
        ${CMAKE_CURRENT_SOURCE_DIR}/src/Hardware/libcana.so 
        ${CMAKE_CURRENT_SOURCE_DIR}/src/libcan/libcan.a 
        ${CMAKE_CURRENT_SOURCE_DIR}/src/App/libkey.so
        kdl_parser
    )
endif()

install(TARGETS follow_1
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(TARGETS follow_1_pos
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(TARGETS arm_control
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
)

install(DIRECTORY
  ${CMAKE_CURRENT_SOURCE_DIR}/launch
  ${CMAKE_CURRENT_SOURCE_DIR}/models
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
catkin_install_python(PROGRAMS scripts/sample.py
                              scripts/mid_image_publish.py
                              scripts/left_image_publish.py
                              scripts/right_image_publish.py
                              scripts/action_replay.py
                              scripts/back_to_origin.py
                              scripts/Subscriber.py
                              scripts/video_check.py
                              scripts/pos_publisher.py
                              scripts/linear_interpolation.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
