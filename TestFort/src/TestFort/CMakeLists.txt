cmake_minimum_required(VERSION 3.10.2)

#project(Terrain LANGUAGES Fortran)
enable_language(Fortran)

file(GLOB_RECURSE sources *.f90 *.f *.fi)

if(CMAKE_Linux)
list(REMOVE_ITEM sources ${CMAKE_CURRENT_SOURCE_DIR}/operating_sys_windows.f)
elseif(CMAKE_Windows)
list(REMOVE_ITEM sources ${CMAKE_CURRENT_SOURCE_DIR}/operating_sys_linux.f)
endif()

add_executable(TestFort ${sources})
