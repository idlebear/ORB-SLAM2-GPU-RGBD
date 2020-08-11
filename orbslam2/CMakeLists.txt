cmake_minimum_required(VERSION 3.0.2)
project(orbslam2)

IF(NOT ROS_BUILD_TYPE)
  SET(ROS_BUILD_TYPE Release)
ENDIF()

MESSAGE(STATUS "Build type: " ${ROS_BUILD_TYPE})

## Compile as C++11, supported in ROS Kinetic and newer
include(FindOpenMP)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS} -Wall  -O3 ") #-march=native ")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS} -Wall  -O3") # -march=native")
set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")

# Check C++11 or C++0x support
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -std=c++11")
add_definitions(-DCOMPILEDWITHC11)
message(STATUS "Using flag -std=c++11.")

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED roscpp cv_bridge sensor_msgs std_msgs message_filters)

find_package(CUDA QUIET REQUIRED)
set(CUDA_PROPAGATE_HOST_FLAGS OFF)
SET(CUDA_HOST_COMPILER /usr/bin/g++)
LIST(APPEND CUDA_NVCC_FLAGS "--compiler-options -fno-strict-aliasing -use_fast_math")

# LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/../../../cmake_modules)
# MESSAGE( ${CMAKE_MODULE_PATH} )

find_package(OpenCV 4.4 REQUIRED) 
find_package(Eigen3 REQUIRED)
find_package(Pangolin REQUIRED)
find_package(realsense2 REQUIRED)
find_package(Boost REQUIRED)

message(STATUS "Building from " ${PROJECT_SOURCE_DIR})
message(STATUS ${OpenCV_INCLUDE_DIRS})
message(STATUS ${OpenCV_LIBRARIES})
message(STATUS ${CUDA_INCLUDE_DIRS})
message(STATUS ${EIGEN3_INCLUDE_DIRS})
message(STATUS ${realsense2_INCLUDE_DIR})


################################################
## Declare ROS dynamic reconfigure parameters ##
################################################

## To declare and build dynamic reconfigure parameters within this
## package, follow these steps:
## * In the file package.xml:
##   * add a build_depend and a exec_depend tag for "dynamic_reconfigure"
## * In this file (CMakeLists.txt):
##   * add "dynamic_reconfigure" to
##     find_package(catkin REQUIRED COMPONENTS ...)
##   * uncomment the "generate_dynamic_reconfigure_options" section below
##     and list every .cfg file to be processed

## Generate dynamic reconfigure parameters in the 'cfg' folder
# generate_dynamic_reconfigure_options(
#   cfg/DynReconf1.cfg
#   cfg/DynReconf2.cfg
# )

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if your package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES orbslam2
#  CATKIN_DEPENDS other_catkin_pkg
#  DEPENDS system_lib
)

###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(
# include
  ${catkin_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${PROJECT_SOURCE_DIR}
  ${PROJECT_SOURCE_DIR}/../
  ${PROJECT_SOURCE_DIR}/../include
  ${OpenCV_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIRS}
  ${Pangolin_INCLUDE_DIRS}
  ${realsense2_INCLUDE_DIR}
  # nvToolsExt
) 

## Declare a C++ library
# add_library(${PROJECT_NAME}
#   src/${PROJECT_NAME}/orbslam2.cpp
# )


cuda_include_directories(
  ${CUDA_TOOLKIT_ROOT_DIR}/samples/common/inc
)

cuda_add_library(${PROJECT_NAME} SHARED
  ${PROJECT_SOURCE_DIR}/../src/System.cc
  ${PROJECT_SOURCE_DIR}/../src/Tracking.cc
  ${PROJECT_SOURCE_DIR}/../src/LocalMapping.cc
  ${PROJECT_SOURCE_DIR}/../src/LoopClosing.cc
  ${PROJECT_SOURCE_DIR}/../src/ORBextractor.cc
  ${PROJECT_SOURCE_DIR}/../src/ORBmatcher.cc
  ${PROJECT_SOURCE_DIR}/../src/FrameDrawer.cc
  ${PROJECT_SOURCE_DIR}/../src/Converter.cc
  ${PROJECT_SOURCE_DIR}/../src/MapPoint.cc
  ${PROJECT_SOURCE_DIR}/../src/KeyFrame.cc
  ${PROJECT_SOURCE_DIR}/../src/Map.cc
  ${PROJECT_SOURCE_DIR}/../src/MapDrawer.cc
  ${PROJECT_SOURCE_DIR}/../src/Optimizer.cc
  ${PROJECT_SOURCE_DIR}/../src/PnPsolver.cc
  ${PROJECT_SOURCE_DIR}/../src/Frame.cc
  ${PROJECT_SOURCE_DIR}/../src/KeyFrameDatabase.cc
  ${PROJECT_SOURCE_DIR}/../src/Sim3Solver.cc
  ${PROJECT_SOURCE_DIR}/../src/Initializer.cc
  ${PROJECT_SOURCE_DIR}/../src/Viewer.cc
  ${PROJECT_SOURCE_DIR}/../src/cuda/Allocator_gpu.cu
  ${PROJECT_SOURCE_DIR}/../src/cuda/Fast_gpu.cu
  ${PROJECT_SOURCE_DIR}/../src/cuda/Orb_gpu.cu
  ${PROJECT_SOURCE_DIR}/../src/cuda/Cuda.cu
)

set(LIBS 
  ${catkin_LIBRARIES}
  ${OpenCV_LIBRARIES} 
  ${CUDA_LIBRARIES}
  ${EIGEN3_LIBRARIES}
  ${Pangolin_LIBRARIES}
  ${realsense2_LIBRARIES}
  ${Boost_LIBRARIES}
  ${PROJECT_SOURCE_DIR}/../Thirdparty/DBoW2/lib/libDBoW2.so
  ${PROJECT_SOURCE_DIR}/../Thirdparty/g2o/lib/libg2o.so
  ${PROJECT_SOURCE_DIR}/../build/lib/libORB_SLAM2.so
)


## Add cmake target dependencies of the library
## as an example, code may need to be generated before libraries
## either from message generation or dynamic reconfigure
# add_dependencies(${PROJECT_NAME} ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

## Declare a C++ executable
## With catkin_make all packages are built within a single CMake context
## The recommended prefix ensures that target names across packages don't collide
# add_executable(${PROJECT_NAME}_node src/orbslam2_node.cpp)

## Rename C++ executable without prefix
## The above recommended prefix causes long target names, the following renames the
## target back to the shorter version for ease of user use
## e.g. "rosrun someones_pkg node" instead of "rosrun someones_pkg someones_pkg_node"
# set_target_properties(${PROJECT_NAME}_node PROPERTIES OUTPUT_NAME node PREFIX "")

## Add cmake target dependencies of the executable
## same as for the library above
# add_dependencies(${PROJECT_NAME}_node ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})


## Specify libraries to link a library or executable target against
# target_link_libraries(${PROJECT_NAME}_node
#   ${catkin_LIBRARIES}
# )

# Csi camera publishing 
add_executable(csi
    ${PROJECT_SOURCE_DIR}/../Examples/ROS/ORB_SLAM2/src/csi_camera.cc
)

target_link_libraries(csi
    ${LIBS}
)

# Node for monocular camera
add_executable(mono
    ${PROJECT_SOURCE_DIR}/../Examples/ROS/ORB_SLAM2/src/ros_mono.cc
)

target_link_libraries(mono
    ${LIBS}
)

# Node for stereo camera
add_executable(stereo
    ${PROJECT_SOURCE_DIR}/../Examples/ROS/ORB_SLAM2/src/ros_stereo.cc
)

target_link_libraries(stereo
    ${LIBS}
)

# Node for RGB-D camera
add_executable(rgbd
    ${PROJECT_SOURCE_DIR}/../Examples/ROS/ORB_SLAM2/src/ros_rgbd.cc
)

target_link_libraries(rgbd
    ${LIBS}
)

# real sense rgbd camera publishing 
add_executable(real_sense_rgbd
    ${PROJECT_SOURCE_DIR}/../Examples/ROS/ORB_SLAM2/src/rgbd_real_sense_ros.cc
)

target_link_libraries(real_sense_rgbd
    ${LIBS}
)



#############
## Install ##
#############

# all install targets should use catkin DESTINATION variables
# See http://ros.org/doc/api/catkin/html/adv_user_guide/variables.html

## Mark executable scripts (Python etc.) for installation
## in contrast to setup.py, you can choose the destination
# catkin_install_python(PROGRAMS
#   scripts/my_python_script
#   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark executables for installation
## See http://docs.ros.org/melodic/api/catkin/html/howto/format1/building_executables.html
install(TARGETS mono stereo rgbd csi real_sense_rgbd
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

## Mark libraries for installation
## See http://docs.ros.org/melodic/api/catkin/html/howto/format1/building_libraries.html
# install(TARGETS ${PROJECT_NAME}
#   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
# )

## Mark cpp header files for installation
# install(DIRECTORY include/${PROJECT_NAME}/
#   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
#   FILES_MATCHING PATTERN "*.h"
#   PATTERN ".svn" EXCLUDE
# )

## Mark other files for installation (e.g. launch and bag files, etc.)
# install(FILES
#   # myfile1
#   # myfile2
#   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
# )

install(DIRECTORY launch
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

#############
## Testing ##
#############

## Add gtest based cpp test target and link libraries
# catkin_add_gtest(${PROJECT_NAME}-test test/test_orbslam2.cpp)
# if(TARGET ${PROJECT_NAME}-test)
#   target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME})
# endif()

## Add folders to be run by python nosetests
# catkin_add_nosetests(test)
