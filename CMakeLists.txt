cmake_minimum_required(VERSION 3.0.0 FATAL_ERROR)

################### Variables. ####################
# Change if you want modify path or other values. #
###################################################

set(PROJECT_NAME LocalExpansionStereo)
# Output Variables
set(OUTPUT_DEBUG Debug/bin)
set(OUTPUT_RELEASE Release/bin)
# Folders files
set(CPP_DIR_1 ./${PROJECT_NAME})
set(HEADER_DIR_1 ./${PROJECT_NAME})

############## CMake Project ################
#        The main options of project        #
#############################################

project(${PROJECT_NAME} CXX)

# Define Release by default.
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release")
  message(STATUS "Build type not specified: Use Release by default.")
endif(NOT CMAKE_BUILD_TYPE)

############## Artefacts Output #################
# Defines outputs , depending Debug or Release. #
#################################################

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${OUTPUT_DEBUG}")
  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${OUTPUT_DEBUG}")
  set(CMAKE_EXECUTABLE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${OUTPUT_DEBUG}")
else()
  set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${OUTPUT_REL}")
  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${OUTPUT_REL}")
  set(CMAKE_EXECUTABLE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${OUTPUT_REL}")
endif()

################### Dependencies ##################
# Add Dependencies to project.                    #
###################################################

option(BUILD_DEPENDS
   "Build other CMake project."
   ON
)

# Dependencies : disable BUILD_DEPENDS to link with lib already build.
if(BUILD_DEPENDS)
   add_subdirectory(maxflow ${CMAKE_BINARY_DIR}/maxflow)
else()
   link_directories(dependencies/maxflow/build/)
endif()

find_package(OpenCV REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS})

if(APPLE)
  find_package(LLVM HINTS /usr/local/opt/llvm/lib/cmake/llvm)
  set(CMAKE_C_COMPILER ${LLVM_TOOLS_BINARY_DIR}/clang)
  set(CMAKE_CXX_COMPILER ${LLVM_TOOLS_BINARY_DIR}/clang++)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fopenmp")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp")
endif()

################# Flags ################
# Defines Flags for Windows and Linux. #
########################################

if(MSVC)
   set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /W3 /MD /Od /EHsc")
   set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /W3 /GL /Od /Oi /Gy /EHsc")
endif(MSVC)
if(NOT MSVC)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
   if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
       set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
   endif()
endif(NOT MSVC)

################ Files ################
#   --   Add files to project.   --   #
#######################################

file(GLOB SRC_FILES
    ${CPP_DIR_1}/*.cpp
    ${HEADER_DIR_1}/*.h
)

# Add executable to build.
add_executable(${PROJECT_NAME}
   ${SRC_FILES}
)

# Link with other dependencies.
target_link_libraries(${PROJECT_NAME} maxflow ${OpenCV_LIBS} ${OpenMP_LIBS})
