cmake_minimum_required(VERSION 3.0.2)
project(cv_bridge)

find_package(catkin REQUIRED COMPONENTS rosconsole sensor_msgs)

if(NOT ANDROID)
  find_package(PythonLibs)

  if(PYTHONLIBS_VERSION_STRING VERSION_LESS "3.8")
    # Debian Buster
    find_package(Boost REQUIRED python37)
  else()
    # Ubuntu Focal
    find_package(Boost REQUIRED python)
  endif()
else()
find_package(Boost REQUIRED)
endif()

####################################################################################################
# set(_opencv_version 4)
# find_package(OpenCV 4 QUIET)
# if(NOT OpenCV_FOUND)
#   message(STATUS "Did not find OpenCV 4, trying OpenCV 3")
#   set(_opencv_version 3)
# endif()
# find_package(OpenCV ${_opencv_version} REQUIRED
#   COMPONENTS
#     opencv_core
#     opencv_imgproc
#     opencv_imgcodecs
#   CONFIG
# )
####################################################################################################

####################################################################################################
set(_opencv_version 4.10.0)
set(OpenCV_DIR "/opt/opencv-${_opencv_version}/lib/cmake/opencv4")
# set(OpenCV_DIR "/usr/lib/aarch64-linux-gnu/cmake/opencv4")
find_package(OpenCV ${_opencv_version} EXACT REQUIRED CONFIG 
    PATHS ${OpenCV_DIR}
    NO_DEFAULT_PATH
    NO_CMAKE_FIND_ROOT_PATH
)
message(STATUS "${PROJECT_NAME} OpenCV_VERSION           : ${OpenCV_VERSION}")
message(STATUS "${PROJECT_NAME} OpenCV_INCLUDE_DIRS      : ${OpenCV_INCLUDE_DIRS}")
message(STATUS "${PROJECT_NAME} OpenCV_LIBRARIES         : ${OpenCV_LIBRARIES}")
message(STATUS "${PROJECT_NAME} OpenCV_LIBRARIES details : ")
foreach(component ${OpenCV_LIBRARIES})
    if(TARGET ${component})
        message(STATUS " |  OpenCV target : ${component}")

        get_target_property(lib_type ${component} TYPE)
        message(STATUS " |   |                           TYPE : ${lib_type}")

        get_target_property(lib_loc ${component} IMPORTED_LOCATION)
        message(STATUS " |   |              IMPORTED_LOCATION : ${lib_loc}")

        get_target_property(lib_loc_debug ${component} IMPORTED_LOCATION_DEBUG)
        message(STATUS " |   |        IMPORTED_LOCATION_DEBUG : ${lib_loc_debug}")

        get_target_property(lib_loc_release ${component} IMPORTED_LOCATION_RELEASE)
        message(STATUS " |   |      IMPORTED_LOCATION_RELEASE : ${lib_loc_release}")

        get_target_property(include_dirs ${component} INTERFACE_INCLUDE_DIRECTORIES)
        message(STATUS " |   |  INTERFACE_INCLUDE_DIRECTORIES : ${include_dirs}")
    else()
        message(STATUS " |  OpenCV library : ${component}")
    endif()
endforeach()
####################################################################################################

####################################################################################################
# catkin_package(
#   INCLUDE_DIRS include
#   LIBRARIES ${PROJECT_NAME}
#   CATKIN_DEPENDS rosconsole sensor_msgs
#   DEPENDS OpenCV
#   CFG_EXTRAS cv_bridge-extras.cmake
# )
####################################################################################################

####################################################################################################
catkin_package(
  INCLUDE_DIRS include ${OpenCV_INCLUDE_DIRS}
  LIBRARIES ${PROJECT_NAME} ${OpenCV_LIBRARIES}
  CATKIN_DEPENDS rosconsole sensor_msgs
  DEPENDS OpenCV
  CFG_EXTRAS cv_bridge-extras.cmake
)
####################################################################################################

catkin_python_setup()

include_directories(include ${Boost_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS} ${catkin_INCLUDE_DIRS})
link_directories(${Boost_LIBRARY_DIRS})

if(NOT ANDROID)
add_subdirectory(python)
endif()
add_subdirectory(src)
if(CATKIN_ENABLE_TESTING)
  add_subdirectory(test)
endif()

# install the include folder
install(
  DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)
