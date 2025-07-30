cmake_minimum_required(VERSION 3.0.2)
project(numpy_eigen)

set(CMAKE_CXX_STANDARD 14)

find_package(catkin REQUIRED COMPONENTS python_module)
find_package(Boost REQUIRED COMPONENTS system)
find_package(Eigen3 REQUIRED)

catkin_python_setup()

add_definitions(${EIGEN_DEFINITIONS})
include_directories(include ${EIGEN3_INCLUDE_DIRS} ${catkin_INCLUDE_DIRS})
include_directories("${PROJECT_SOURCE_DIR}/include/numpy_eigen")

catkin_package(
  INCLUDE_DIRS include ${catkin_INCLUDE_DIRS}
  LIBRARIES ${PROJECT_NAME}
  DEPENDS Boost
  CFG_EXTRAS numpy_eigen-extras.cmake
)
add_definitions(-D__STRICT_ANSI__)

# ignore "error: return-statement with a value, in function returning ‘void’" in "__multiarray_api.h"
add_compile_options(-fpermissive)

include_directories(include ${Boost_INCLUDE_DIRS})

IF(APPLE)
  link_directories( /usr/local/lib )
ENDIF(APPLE)

# Add numpy headers explicitly to handle pip installations as well
find_package(Python3 COMPONENTS Interpreter Development REQUIRED)
execute_process(
  COMMAND ${Python3_EXECUTABLE} -c "import numpy; import os; print(os.path.join(os.path.dirname(numpy.__file__), 'core', 'include', 'numpy'))"
  OUTPUT_VARIABLE NUMPY_INCLUDE_PATH
  OUTPUT_STRIP_TRAILING_WHITESPACE
  ERROR_QUIET
)

if(NUMPY_INCLUDE_PATH AND EXISTS ${NUMPY_INCLUDE_PATH})
  message("Found numpy headers at ${NUMPY_INCLUDE_PATH}")
  include_directories(${NUMPY_INCLUDE_PATH})
  include_directories(${NUMPY_INCLUDE_PATH}/..)
endif()

##############
## Building ##
##############

INCLUDE(src/autogen_files.cmake)
add_python_export_library(${PROJECT_NAME} 
  ${PROJECT_SOURCE_DIR}/python/${PROJECT_NAME} ${AUTOGEN_FILES}
)

target_link_libraries(${PROJECT_NAME} 
                      ${catkin_LIBRARIES} ${Boost_LIBRARIES})
                      
INCLUDE(src/autogen_test_files.cmake)
add_python_export_library(${PROJECT_NAME}_test 
      ${PROJECT_SOURCE_DIR}/python/${PROJECT_NAME} ${AUTOGEN_TEST_FILES}
  )

#############
## Testing ##
#############
if(CATKIN_ENABLE_TESTING)
  ## Add nosetest based cpp test target.
  catkin_add_nosetests(test/numpy_eigen_tests.py)
endif()

install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

##################
## Installation ##
##################

install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

## Mark cpp header files for installation
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)
