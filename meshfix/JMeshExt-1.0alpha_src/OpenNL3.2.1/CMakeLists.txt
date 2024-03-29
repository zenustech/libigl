######################################
#      CMake version checks
######################################
if(COMMAND cmake_minimum_required)
   cmake_minimum_required(VERSION 2.4)
endif(COMMAND cmake_minimum_required)
include_directories(SuperLU/SRC)

if(COMMAND cmake_policy)
	if(POLICY CMP0003)
		cmake_policy(SET CMP0003 OLD)
	endif(POLICY CMP0003)
	if(POLICY CMP0005)
		cmake_policy(SET CMP0005 OLD)
	endif(POLICY CMP0005)
	if(POLICY CMP0011)
		cmake_policy(SET CMP0011 OLD) # or NEW
	endif(POLICY CMP0011)
endif(COMMAND cmake_policy)


######################################
#              Project
######################################
PROJECT(OpenNL3)
SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

SET(GEEX_BUILD_PATH ${PROJECT_BINARY_DIR})
SET(BINARIES_RELATIVE_PATH /binaries/bin/)

IF (WIN32)
    SET(LIBRARIES_RELATIVE_PATH /binaries/bin/)
ELSE(WIN32)
    SET(LIBRARIES_RELATIVE_PATH /binaries/lib/)
ENDIF(WIN32)

IF(WIN32)
    SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}${BINARIES_RELATIVE_PATH})
ELSE(WIN32)
    SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}${LIBRARIES_RELATIVE_PATH})
ENDIF(WIN32)

SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}${BINARIES_RELATIVE_PATH})


IF (EXISTS ${CMAKE_SOURCE_DIR}/CMakeOptions.txt)
   MESSAGE(INFO: "Using local options file: " ${CMAKE_SOURCE_DIR}/CMakeOptions.txt)
   INCLUDE(${CMAKE_SOURCE_DIR}/CMakeOptions.txt)
ENDIF(EXISTS ${CMAKE_SOURCE_DIR}/CMakeOptions.txt)

IF(USE_CNC)
    ADD_DEFINITIONS(-DNL_USE_CNC)
    
    IF(CUDA_BUILD_EMULATION)
        ADD_DEFINITIONS(-D__DEVICE_EMULATION__)
    ENDIF(CUDA_BUILD_EMULATION)


    IF(WIN32)
        ADD_DEFINITIONS(-DOS_WIN)
    ELSE(WIN32)
        ADD_DEFINITIONS(-DOS_LINUX)
    ENDIF(WIN32)

    INCLUDE(${CMAKE_SOURCE_DIR}/CMakeModules/FindCUDA.cmake)
    
    IF (CMAKE_BUILD_TYPE MATCHES Debug)
        SET(CUDA_COMPILE_TIME_EXTRA_FLAGS -G)
    ENDIF (CMAKE_BUILD_TYPE MATCHES Debug)
    
    IF(CNC_OPTIMIZE_FOR_THIS_MACHINE)
        IF(COMMAND CUDA_DISCOVER_DEVICE_FLAGS)
        ELSE(COMMAND CUDA_DISCOVER_DEVICE_FLAGS)
	        TRY_COMPILE(COMPILE_OK ${PROJECT_BINARY_DIR}/CMake ${PROJECT_SOURCE_DIR}/CMake CUDA_DISCOVER_DEVICE_FLAGS)
	        IF(NOT COMPILE_OK)
		        IF(WIN32)
			        MESSAGE("error compiling CUDA_DISCOVER_DEVICE_FLAGS extention")
		        ELSE(WIN32)
			        MESSAGE(FATAL_ERROR "error compiling CUDA_DISCOVER_DEVICE_FLAGS extention")
		        ENDIF(WIN32)
	        ENDIF(NOT COMPILE_OK)
	        LOAD_COMMAND(CUDA_DISCOVER_DEVICE_FLAGS ${PROJECT_BINARY_DIR}/CMake ${PROJECT_BINARY_DIR}/CMake/Debug)
        ENDIF(COMMAND CUDA_DISCOVER_DEVICE_FLAGS)
        CUDA_DISCOVER_DEVICE_FLAGS()
        
        SET(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} ${CUDA_DEVICE_FLAGS} )
        ADD_DEFINITIONS(${CUDA_DEVICE_FLAGS})
    ELSE(CNC_OPTIMIZE_FOR_THIS_MACHINE)
      SET(CUDA_NVCC_FLAGS ${CNC_NVCC_FLAGS})
    ENDIF(CNC_OPTIMIZE_FOR_THIS_MACHINE)
    
ENDIF(USE_CNC)

IF (CMAKE_BUILD_TYPE MATCHES Debug)
    ADD_DEFINITIONS(-DNL_DEBUG)
    IF(PARANOID_DEBUG)
        ADD_DEFINITIONS(-DNL_PARANOID)
    ENDIF(PARANOID_DEBUG)
ENDIF (CMAKE_BUILD_TYPE MATCHES Debug)


#########################################
# ADD HERE ADDITIONAL INCLUDE DIRECTORIES
#########################################
INCLUDE_DIRECTORIES(${CMAKE_SOURCE_DIR})
INCLUDE_DIRECTORIES(src)

#########################################
# ADD HERE SUBDIRECTORIES
#########################################
SET(USE_SUPERLU TRUE)
# relative to src/
SET(SUPERLU_HOME ../SuperLU)
SET(SUPERLU_NAME superlu)
set(SUPERLU_BUILD_EXAMPLES OFF)
set(SUPERLU_ENABLE_TESTING OFF)
include_directories("${SuperLU}/src") 
ADD_SUBDIRECTORY(SuperLU)
ADD_SUBDIRECTORY(src)

#IF(BUILD_EXAMPLES)
#    ADD_SUBDIRECTORY(examples)
#ENDIF(BUILD_EXAMPLES)
