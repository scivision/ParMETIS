cmake_minimum_required(VERSION 3.16...4.1)

if(NOT CMAKE_BUILD_TYPE AND NOT DEFINED ENV{CMAKE_BUILD_TYPE})
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Debug or Release")
endif()

project(parmetis LANGUAGES C VERSION 1.0.0)

# --- need METIS first
include(GNUInstallDirs)
include(cmake/GitSubmodule.cmake)

if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT AND metis_IS_TOP_LEVEL)
  set(CMAKE_INSTALL_PREFIX "${PROJECT_BINARY_DIR}/local" CACHE PATH "install prefix" FORCE)
endif()

message(STATUS "${PROJECT_NAME} ${PROJECT_VERSION} CMake ${CMAKE_VERSION} install prefix ${CMAKE_INSTALL_PREFIX}")

git_submodule(${CMAKE_CURRENT_SOURCE_DIR}/METIS)
add_subdirectory(METIS)

set(CMAKE_C_STANDARD 99)

add_compile_definitions(
"REALTYPEWIDTH=$<IF:$<BOOL:${REALTYPEWIDTH}>,${REALTYPEWIDTH},32>"
"IDXTYPEWIDTH=$<IF:$<BOOL:${IDXTYPEWIDTH}>,${IDXTYPEWIDTH},32>"
)

find_package(MPI REQUIRED)

include(./conf/gkbuild.cmake)

install(FILES include/parmetis.h TYPE INCLUDE)

add_library(parmetis)
target_include_directories(parmetis PUBLIC
"$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include;${PROJECT_SOURCE_DIR}/libparmetis;${PROJECT_SOURCE_DIR}/METIS/include>"
$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)
target_link_libraries(parmetis PRIVATE MPI::MPI_C metis::metis)

if(SHARED)
  target_link_libraries(parmetis PRIVATE metis GKlib)
endif()

add_library(parmetis::parmetis INTERFACE IMPORTED GLOBAL)
target_link_libraries(parmetis::parmetis INTERFACE parmetis)

install(TARGETS parmetis EXPORT ${PROJECT_NAME}-targets)

add_subdirectory(libparmetis)
add_subdirectory(programs)

# This is for testing during development and is not being distributed
#add_subdirectory(test)

include(cmake/install.cmake)

file(GENERATE OUTPUT .gitignore CONTENT "*")
