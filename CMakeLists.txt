cmake_minimum_required(VERSION 3.5)
project(segwayrmp)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fpic  -Wl,-R${PROJECT_SOURCE_DIR}/lib")
set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -fpic")
# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
  add_compile_options(-Werror)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclpy REQUIRED)
find_package(std_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(segway_msgs REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(rclcpp_action REQUIRED)
find_package(rclcpp_components REQUIRED)

if (${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64")
  set(arch_suffix x86_64)
elseif (${CMAKE_SYSTEM_PROCESSOR} STREQUAL "aarch64")
  set(arch_suffix arm64-v8a)
else()
  message(FATAL_ERROR "unsupported architecture")
endif()

add_executable(SmartCar
  src/SmartCar.cpp
  src/robot.cpp
)
target_include_directories(SmartCar PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)
target_link_directories(SmartCar PUBLIC
  ${CMAKE_CURRENT_SOURCE_DIR}/lib
)
# message(WARNING ${PROJECT_SOURCE_DIR} + "!!" + ${CMAKE_CURRENT_SOURCE_DIR} + "!!" + ${PROJECT_NAME})
ament_target_dependencies(SmartCar
  "rclcpp" "std_msgs" "nav_msgs" "sensor_msgs" "geometry_msgs" "segway_msgs" "tf2" "tf2_ros"
)
target_link_libraries(SmartCar
  ${PROJECT_SOURCE_DIR}/lib/libctrl_${arch_suffix}.so
)

add_executable(drive_segway_sample
  tools/drive_segway_sample.cpp
)
target_include_directories(drive_segway_sample PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)
ament_target_dependencies(drive_segway_sample
  "rclcpp" "std_msgs" "nav_msgs" "sensor_msgs" "geometry_msgs" "segway_msgs" "tf2" "tf2_ros"
) 

install(TARGETS
  SmartCar
  drive_segway_sample
  DESTINATION
  lib/${PROJECT_NAME})

install(FILES
        ${PROJECT_SOURCE_DIR}/lib/libctrl_${arch_suffix}.so
        DESTINATION lib)

install(PROGRAMS
        ${PROJECT_SOURCE_DIR}/lib/ctrl_${arch_suffix}
        DESTINATION bin)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # uncomment the line when a copyright and license is not present in all source files
  #set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # uncomment the line when this package is not in a git repo
  #set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
