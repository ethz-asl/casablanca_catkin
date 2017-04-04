cmake_minimum_required(VERSION 2.8.3)
project(casablanca_catkin)

find_package(catkin_simple REQUIRED)
catkin_simple()

include_directories(${Eigen_INCLUDE_DIRS})

include(ExternalProject)

# The configure step fails at catkin_package() if this directory
# doesn't exist yet. Create it at configure time!
file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include)

# Patch away the linking to the gomp OpenMP implementation, in case we use the Intel OpenMP implementation and Clang.
if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
	set(GOMP_PATCH "")
else (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
	set(GOMP_PATCH "--dry-run")
endif (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")

ExternalProject_Add(casablanca_src
  GIT_REPOSITORY https://github.com/Microsoft/cpprestsdk.git
  CONFIGURE_COMMAND cd ../casablanca_src/Release/ && cmake -DCMAKE_CXX_FLAGS=-fPIC -DGFLAGS=ON 
    -DCMAKE_INSTALL_PREFIX=${CATKIN_DEVEL_PREFIX} -DBUILD_SHARED_LIBS=ON 
    -DBUILD_DOCUMENTATION=OFF -DCMAKE_VERBOSE_MAKEFILE=ON
	-DCMAKE_BUILD_TYPE:STRING=Release
    -DBUILD_EXAMPLES=OFF
    -DBUILD_TESTING=OFF
    -DEXPORT_NONPUBLIC_HEADERS=ON
  BUILD_COMMAND cd ../casablanca_src/Release/ && make -j8
  INSTALL_COMMAND cd ../casablanca_src/Release/ && make install -j8
)

cs_add_library(${PROJECT_NAME} src/dependency_tracker.cc)
add_dependencies(${PROJECT_NAME} casablanca_src)
target_link_libraries(${PROJECT_NAME}
  ${CATKIN_DEVEL_PREFIX}/lib/libcpprest${CMAKE_SHARED_LIBRARY_SUFFIX})

cs_install()

cs_export(INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
          CFG_EXTRAS casablanca-extras.cmake)
