cmake_minimum_required(VERSION 2.8.3)
project(sweep_ros)



find_package(catkin REQUIRED COMPONENTS
        rosconsole
        roscpp
        sensor_msgs
        pcl_msgs
        )

catkin_package()

find_package(PCL REQUIRED)
# find_package(Sweep REQUIRED)
############################
# Sweep SDK installation:
set(SWEEP_VERSION_MAJOR 1)
set(SWEEP_VERSION_MINOR 1)
set(SWEEP_VERSION_PATCH 1)

# Platform specific compiler and linker options.
set(libsweep_OS unix)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/sweep-sdk/libsweep/cmake/config.h.in
               ${CMAKE_CURRENT_SOURCE_DIR}/include/sweep/config.h)


# C++11 requires us to link in pthreads and equivalents.
find_package(Threads REQUIRED)

# libsweep target.
file(GLOB libsweep_OS_SOURCES sweep-sdk/libsweep/src/${libsweep_OS}/*.cc)

set(libsweep_IMPL_SOURCES sweep-sdk/libsweep/src/sweep.cc)

set(libsweep_SOURCES ${libsweep_OS_SOURCES} ${libsweep_IMPL_SOURCES} sweep-sdk/libsweep/src/protocol.cc)
file(GLOB libsweep_HEADERS sweep-sdk/libsweep/include/*.h sweep-sdk/libsweep/include/sweep/*.h sweep-sdk/libsweep/include/sweep/*.hpp)

add_library(sweep SHARED ${libsweep_SOURCES} ${libsweep_HEADERS})
target_include_directories(sweep PRIVATE include sweep-sdk/libsweep/include/sweep ${CMAKE_CURRENT_BINARY_DIR}/include)
target_link_libraries(sweep ${CMAKE_THREAD_LIBS_INIT})

set_property(TARGET sweep PROPERTY VERSION "${SWEEP_VERSION_MAJOR}.${SWEEP_VERSION_MINOR}.${SWEEP_VERSION_PATCH}")
set_property(TARGET sweep PROPERTY SOVERSION "${SWEEP_VERSION_MAJOR}")

############################

# find_package(Threads REQUIRED)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99 -Wall -Wextra")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Wextra")

include_directories(
        include ${catkin_INCLUDE_DIRS}
        ${PCL_INCLUDE_DIRS}
        sweep-sdk/libsweep/include
        ${CMAKE_CURRENT_SOURCE_DIR}/include
)

## Add link directories
link_directories(
        ${PCL_LIBRARY_DIRS}
)


add_executable(sweep_node src/node.cpp)

target_link_libraries(sweep_node ${catkin_LIBRARIES} sweep)

target_include_directories(sweep_node SYSTEM PRIVATE ${LIBSWEEP_INCLUDE_DIR})

install(TARGETS sweep 
        LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})
install(DIRECTORY sweep-sdk/libsweep/include/sweep 
        DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/include/sweep/config.h 
        DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}/sweep)

install(TARGETS sweep_node
        ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
        )
