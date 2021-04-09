#windows version
#cmake_minimum_required(VERSION 3.16.5)
#linux version
cmake_minimum_required(VERSION 3.10.2)

enable_language(Fortran)
project(TestFortCross VERSION 10.1)

#decide on which operating system you are actually on
if("${CMAKE_SYSTEM_NAME}" STREQUAL "Linux")
message("working on linux")
set(CMAKE_Linux 1)
set(CMAKE_Windows 0)
elseif("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
message("working on windows")
set(CMAKE_Linux 0)
set(CMAKE_Windows 1)
else()
message("undefined OS")
set(CMAKE_Linux 0)
set(CMAKE_Windows 0)
endif()

message("CMAKE_SYSTEM_NAME: ${CMAKE_SYSTEM_NAME}")
configure_file(./CompilationInfo.h.in CompilationInfo.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

#FLAGS ON WINDOWS
if(CMAKE_Windows)
set(CMAKE_GENERATOR "Visual Studio 16 2019")
#set(CMAKE_GENERATOR "Visual Studio 9 2008")
##########################################################################################
#  NOTES on compiler option for Fortran in WindSim core. COMPILING ON WINDOWS WITH IFORT.
#
#
#1)	In Release and Debug mode we need to activate the option to use byte as record lenght.
#	This is needed in reading and writing of binary files (phida and vtf) to be compatible with c++ read and writing.
#	check Fortran - Data (in Visual Studio)
#	compiler option: /assume:byterecl
#
#2)	heap_arrays set at 0 as the setting in Visual Studio. (avoids stack overflow) [optional]
#	check Fortran - Optimizaton (in Visual Studio)
#	compiler option: /heap-arrays0
#
#3)	check Fortran - Libraries - Runtime library (in Visual Studio)
#	compiler options for multithreaded: /libs:static /threads /dbglibs
#	in debug mode we add the /dbglibs option
#
#4)	for the compilation on different Os we use a preprocessor which has to be enables with /fpp (-cpp on Linux)
#
#
#-->-->--> these compiler options should always be set
set(CMAKE_Fortran_FLAGS "/assume:byterecl /heap-arrays0 /libs:static /threads /fpp ")
#set(CMAKE_Fortran_FLAGS "/W1 /nologo")
#
#
#
#5)	In Debug mode normally activates the option to Check Routines Interfaces.
#	We need to deativate it to avoid errors due to some wrong interfaces usage.
#	check Fortran - Diagnostics (in Visual Studio)
#	compiler option: /warn:interfaces
#
#6)	In Debug mode compiler option related to runtime should be disabled 
#	check stack frame and check array string bounds rise Access violation errors.
#	the responsible are some functions that are passing a long string to a function that is supposed to read a shorter one or similar.
#	(errors rising while deleting site compliance files and in func_integer2textstring function site compliance function while reading the energy log)
#	check Fortran - Run-time (in Visual Studio)
#	compiler options: /check:bounds /check:stack /check:shape /traceback
#	
#	
#	OPTIONS 5) and 6) should be enabled in the future in debug mode
#
#
#7)	In Release and Debug the option related to Floating Point can modify sligtly the results
#	We have tested some settings and found the best configuration with “/fp-strict” for the Terrain built and “/fp-fast” for the others.
#	The new compiler introduces many more options then the old one and we cannot test all.
#	See this deeper material and the last table about the compiler options. 
#	https://software.intel.com/sites/default/files/managed/3b/37/FP_Consistency_121918.pdf
#	check Fortran - Floatinng Point (in Visual Studio)
#	compiler options: /fp-strict , /fp-fast
#	
#8)	/W.. flags control the warning level of the output, e.g. /W1 shows severe warnings /Wall shows all. We choose /Wall to be the default in Debug mode.
#
#
#FOR DEBUG MODE ONLY --> APPEND:
#/dbglibs and /W1 is default
#set(flag "/dbglibs /Wall") 
#set(CMAKE_Fortran_FLAGS_DEBUG "${CMAKE_Fortran_FLAGS_DEBUG} ${flag}")
#
#FOR RELEASE MODE ONLY --> APPEND:
#/O2 /DNDEBUG is default
#maximize speed
#set(flag "/O2")
set(CMAKE_Fortran_FLAGS_RELEASE "/O2")
#maximize speed plus higher level optimization 
#set(flag "/O3") 
#set(CMAKE_Fortran_FLAGS_RELEASE "${CMAKE_Fortran_FLAGS_RELEASE} ${flag}")
#
#
#################################################################################################

#set(flag "/fp:strict") 
#set(flag "/fp:fast") 
#set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} ${flag}")
#--> check automatic flags set by VS
endif()

#FLAGS ON LINUX
#/fpp - run the fortran preprocessor on source files before they are compiled for gfortran this command is -cpp
#-fdollar-ok doesnt work for a dollor sign as first character
#-fno-range-check avoids checking if the integer is too big for its kind
#-fugly-logint allows to compare logical operands with .eq. instead of .eqv.
if(CMAKE_Linux)
set(CMAKE_GENERATOR "Eclipse CDT4 - Unix Makefiles")
set(flag "-cpp -fno-range-check -fdollar-ok")
set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} ${flag}")
set(default_build_type "Debug")
#set(default_build_type "Release")
set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE STRING "Choose the type of build." FORCE)
endif()

#SHOW ALL USED FLAGS
message("CMAKE_Fortran_FLAGS: ${CMAKE_Fortran_FLAGS}")
message("CMAKE_Fortran_FLAGS_DEBUG: ${CMAKE_Fortran_FLAGS_DEBUG}")
message("CMAKE_Fortran_FLAGS_RELEASE: ${CMAKE_Fortran_FLAGS_RELEASE}")

#show the configuration types that are available by default
message("CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE}")

#Show Fortran Compiler in use
message("CMAKE_Fortran_COMPILER: ${CMAKE_Fortran_COMPILER}")
message("CMAKE_GENERATOR: ${CMAKE_GENERATOR}")

add_subdirectory("./TestFort")
