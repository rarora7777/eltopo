cmake_minimum_required(VERSION 2.8.6)

if (NOT CMAKE_BUILD_TYPE)
  message(STATUS "No build type selected, default to Release")
  set(CMAKE_BUILD_TYPE "Release")
endif()

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/../cmake)

add_definitions(-DNO_GUI)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -g -Wall -Wextra -Wno-reorder -Wno-unknown-pragmas -march=native" )
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
set(LIBELTOPO_INCLUDE_DIR "${CMAKE_CURRENT_LIST_DIR}")
include_directories(
  "${LIBELTOPO_INCLUDE_DIR}/" # because of use of angle bracket #include
  "${LIBELTOPO_INCLUDE_DIR}/../common"
  "${LIBELTOPO_INCLUDE_DIR}/../common/tunicate"
  "${LIBELTOPO_INCLUDE_DIR}/../common/newsparse"
  "${LIBELTOPO_INCLUDE_DIR}/../common/meshes")

find_package(BLAS REQUIRED)
include_directories(${BLAS_INCLUDE_DIR})
find_package(LAPACK REQUIRED)
include_directories(${LAPACK_INCLUDE_DIR})

if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  add_definitions(-DUSE_FORTRAN_BLAS)
  add_definitions(-D__LITTLE_ENDIAN__)
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -flax-vector-conversions")
endif()

if(CMAKE_BUILD_TYPE MATCHES Release)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -funroll-loops")
  set(LIBNAME "eltopo_release")
else()
  set(LIBNAME "eltopo_debug")
endif()



# We need C++11. Put this directive after CGAL's include.
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
endif()

list(APPEND LIBS 
  ${LAPACK_LIBRARIES}
  ${BLAS_LIBRARIES}
)

FILE(GLOB SRCFILES 
  "*.cpp" 
  "../common/tunicate/*.cpp"
  "../common/newsparse/*.cpp"
  "../common/*.cpp"
  )
add_library(${LIBNAME} ${SRCFILES})
add_library(libeltopo ALIAS ${LIBNAME})
set_property(TARGET ${LIBNAME} PROPERTY EXPORT_NAME libeltopo)
target_include_directories(${LIBNAME} SYSTEM INTERFACE 
  $<BUILD_INTERFACE:${LIBELTOPO_INCLUDE_DIR}>
  $<INSTALL_INTERFACE:include>
)

install(TARGETS ${LIBNAME} ARCHIVE DESTINATION ${CMAKE_CURRENT_LIST_DIR})
