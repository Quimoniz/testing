cmake_minimum_required(VERSION 3.7)

PROJECT(tests)

if(POLICY CMP0074)
    cmake_policy(SET CMP0074 NEW)
endif()

SET(CMAKE_CXX_STANDARD 14)
SET(CMAKE_CXX_STANDARD_REQUIRED ON)

SET(ALPAKA_ROOT "${CMAKE_CURRENT_LIST_DIR}/../../alpaka" CACHE STRING "The location of the alpaka library")
LIST(APPEND CMAKE_MODULE_PATH "${ALPAKA_ROOT}")
FIND_PACKAGE("alpaka" REQUIRED)

SET(llama_ROOT "${CMAKE_CURRENT_LIST_DIR}/../../llama" CACHE STRING "The location of the llama library")
find_package("llama" REQUIRED CONFIG)

FIND_PACKAGE(DASH-MPI)

INCLUDE("${ALPAKA_ROOT}/cmake/common.cmake")
#INCLUDE("${ALPAKA_ROOT}/cmake/dev.cmake")

INCLUDE_DIRECTORIES("${gtest_SOURCE_DIR}/include")
INCLUDE_DIRECTORIES("../../gtest/googletest/include")
INCLUDE_DIRECTORIES("../../llama/examples/common")
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_LIST_DIR}/../include)
INCLUDE_DIRECTORIES("${llama_INCLUDE_DIR}")
INCLUDE_DIRECTORIES(${alpaka_INCLUDE_DIRS})
ADD_DEFINITIONS("-D_GLIBCXX_USE_C99=1")
ADD_DEFINITIONS(${alpaka_DEFINITIONS})
add_definitions(${llama_DEFINITIONS})

#ALPAKA_ADD_EXECUTABLE(
#    array_test
#    "array_test.cpp")
#TARGET_LINK_LIBRARIES(
#    array_test
#    PUBLIC "alpaka;gtest_main")
    

#IF(DASH-MPI_FOUND)
#    ALPAKA_ADD_EXECUTABLE(
#        for_each_test
#        "for_each_test.cpp")
#    TARGET_LINK_LIBRARIES(
#        for_each_test
#        PUBLIC "alpaka;${DASH_LIBRARIES};gtest_main")
#ENDIF()

#ALPAKA_ADD_EXECUTABLE(
#    llama_container
#    "llama_container.cpp")
#TARGET_LINK_LIBRARIES(
#    llama_container
#    PUBLIC "${DASH_LIBRARIES}")
#
#ALPAKA_ADD_EXECUTABLE(
#    llama_container_view
#    "llama_container_view.cpp")
#TARGET_LINK_LIBRARIES(
#    llama_container_view
#    PUBLIC "${DASH_LIBRARIES}")

set(LIBRARIES ${LIBRARIES} ${DASH_LIBRARIES})
set(LIBRARIES ${LIBRARIES} ${alpaka_LIBRARIES})
set(INCLUDE_DIRS ${INCLUDE_DIRS} ${alpaka_INCLUDE_DIRS})
set(DEFINITIONS ${DEFINITIONS} ${alpaka_DEFINITIONS})
#set(DEFINITIONS ${DEFINITIONS} ${ALPAKA_DEV_COMPILE_OPTIONS})

# workaround for: NVCC does not incorporate the alpaka defines
if(ALPAKA_ACC_GPU_CUDA_ENABLE OR ALPAKA_ACC_GPU_HIP_ENABLE)
    set(_ALPAKA_COMPILE_DEFINITIONS_CUDA ${_ALPAKA_COMPILE_DEFINITIONS_PUBLIC})
    list_add_prefix("-D" _ALPAKA_COMPILE_DEFINITIONS_CUDA)
    list(APPEND CUDA_NVCC_FLAGS ${_ALPAKA_COMPILE_DEFINITIONS_CUDA})
endif()

ALPAKA_ADD_EXECUTABLE(mephisto-nbody nbody.cpp Dummy.cpp)
target_compile_options(mephisto-nbody PRIVATE ${DEFINITIONS})
target_link_libraries(mephisto-nbody PUBLIC ${LIBRARIES})
