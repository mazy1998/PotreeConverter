cmake_minimum_required(VERSION 3.16)

# Set C++ compiler flags before `project`
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Set the project name.
set(PROJECT_NAME "Converter")
project(${PROJECT_NAME} LANGUAGES C CXX CUDA)

# Set output directories.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin/${CMAKE_BUILD_TYPE}")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin/${CMAKE_BUILD_TYPE}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin/${CMAKE_BUILD_TYPE}")

# Include cmake scripts
include("${CMAKE_CURRENT_SOURCE_DIR}/cmake/cpm.cmake")
include("${CMAKE_CURRENT_SOURCE_DIR}/cmake/dependencies.cmake")

# Set the include directories.
list(APPEND CONVERTER_INCLUDES
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${CMAKE_CURRENT_SOURCE_DIR}/libs
    ${CMAKE_CURRENT_SOURCE_DIR}/modules
    ${tinyply_SOURCE_DIR}/source
)

# Include directories for laszip
include_directories("/home/mazeyar/miniconda3/envs/lod-3dgs/include")
link_directories("/home/mazeyar/miniconda3/envs/lod-3dgs/lib")

# Find and link the Laszip library
find_library(LASZIP_LIB NAMES laszip PATHS "/home/mazeyar/miniconda3/envs/lod-3dgs/lib")

if(LASZIP_LIB)
    message(STATUS "Laszip library found at ${LASZIP_LIB}")
else()
    message(FATAL_ERROR "Laszip library not found!")
endif()

# Set the source files.
aux_source_directory(src SRC_FILES)
aux_source_directory(modules/LasLoader SRC_FILES)
aux_source_directory(modules/tools SRC_FILES)
aux_source_directory(modules/unsuck SRC_FILES)

# Add the executable target
add_executable(${PROJECT_NAME} ${SRC_FILES})

# Link include directories
target_include_directories(${PROJECT_NAME} PRIVATE ${CONVERTER_INCLUDES})

# Set the linking libraries.
list(APPEND CONVERTER_LIBRARIES
    tinyply
    ${PCL_LIBRARIES}  # Assuming PCL is also used
    pthread
    ${LASZIP_LIB}     # Link against the found Laszip library
)

# Link against libraries
target_link_libraries(${PROJECT_NAME} PRIVATE ${CONVERTER_LIBRARIES})