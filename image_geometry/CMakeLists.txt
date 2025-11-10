cmake_minimum_required(VERSION 3.0.2)
project(image_geometry)

find_package(catkin REQUIRED sensor_msgs)

####################################################################################################
# find_package(OpenCV REQUIRED)
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
#         CATKIN_DEPENDS sensor_msgs
#         DEPENDS OpenCV
#         INCLUDE_DIRS include ${OpenCV_INCLUDE_DIRS}
#         LIBRARIES ${PROJECT_NAME}
# )
####################################################################################################

####################################################################################################
catkin_package(
    INCLUDE_DIRS include ${OpenCV_INCLUDE_DIRS}
    LIBRARIES ${PROJECT_NAME} ${OpenCV_LIBRARIES}
    CATKIN_DEPENDS sensor_msgs
    DEPENDS OpenCV
)
####################################################################################################

include_directories(include)
include_directories(${catkin_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS})

# add a library
add_library(${PROJECT_NAME} src/pinhole_camera_model.cpp src/stereo_camera_model.cpp)
target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBRARIES})
add_dependencies(${PROJECT_NAME} ${catkin_EXPORTED_TARGETS})

install(DIRECTORY include/${PROJECT_NAME}/
        DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}/
)

# install library
install(TARGETS ${PROJECT_NAME}
        ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
)

# add tests
if(CATKIN_ENABLE_TESTING)
  add_subdirectory(test)
endif()
