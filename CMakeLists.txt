cmake_minimum_required(VERSION 3.1.0)
project(carto_clion_ws)
set(CMAKE_CXX_STANDARD 11)

# catkin sets this to "on" by default, which causes linking problems
set(BUILD_SHARED_LIBS OFF)

# find GMock globally from here, or else it will complains about duplicated GTest/GMock targets
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/src/cartographer/cmake/modules/")
find_package(GMock REQUIRED)

# use configuration files from the source location instead of the install location
set(CARTOGRAPHER_CONFIGURATION_FILES_DIRECTORY "${CMAKE_SOURCE_DIR}/src/cartographer/configuration_files"
  CACHE PATH ".lua config file directory")

# add the libcartographer directory
add_subdirectory(src/cartographer)

# point the catkin packages to the special Findcartographer.cmake module which is run when find_package(cartographer)
# is called; this module just checks if libcartographer is in the build tree at this point
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/src/cmake/modules/")

# add catkin packages
add_subdirectory(src/cartographer_ros/cartographer_ros_msgs)
add_subdirectory(src/cartographer_ros/cartographer_ros)

# The rviz plugin has to be built as a shared library
set(BUILD_SHARED_LIBS ON)
add_subdirectory(src/cartographer_ros/cartographer_rviz)
add_subdirectory(src/cartographer_ros/docs)
