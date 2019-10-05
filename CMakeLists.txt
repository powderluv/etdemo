cmake_minimum_required(VERSION 3.11)
project(bazel-sucks-edgetpu-cmake-cpp-demo)


set(CMAKE_C_FLAGS "-Wall -pthread ")
set(CMAKE_C_FLAGS_DEBUG "-g -O0")
set(CMAKE_C_FLAGS_RELEASE "-O3")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} -std=c++11  -lstdc++")
set(CMAKE_CXX_FLAGS_DEBUG ${CMAKE_C_FLAGS_DEBUG})
set(CMAKE_CXX_FLAGS_RELEASE ${CMAKE_C_FLAGS_RELEASE})


if(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64")
    set(EDGE_ARCH "k8")
else()
	set(EDGE_ARCH "${CMAKE_SYSTEM_PROCESSOR}")
endif()
message("System ARCH is ${CMAKE_SYSTEM_PROCESSOR} ${EDGE_ARCH}")

#checkout edgetpu
include(ExternalProject)

ExternalProject_Add(edgetpu
    GIT_REPOSITORY https://github.com/google-coral/edgetpu
    GIT_SHALLOW 1
    GIT_PROGRESS 1
    PREFIX "edgetpu"
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    BUILD_BYPRODUCTS libedgetpu.so
    INSTALL_COMMAND cp -f ${CMAKE_BINARY_DIR}/edgetpu/src/edgetpu/libedgetpu/direct/${EDGE_ARCH}/libedgetpu.so.1.0 ${CMAKE_BINARY_DIR}/libedgetpu.so
)

ExternalProject_Add(tf
    GIT_REPOSITORY https://github.com/tensorflow/tensorflow
    GIT_PROGRESS 1
    PREFIX "tensorflow"
    CONFIGURE_COMMAND ./tensorflow/lite/tools/make/download_dependencies.sh
    BUILD_IN_SOURCE 1
    BUILD_COMMAND make -j4 TARGET=generic-aarch64 TARGET_ARCH=armv8-a -C ${CMAKE_BINARY_DIR}/tensorflow/src/tf -f tensorflow/lite/tools/make/Makefile CC=aarch64-unknown-linux-gnu-g++ CXX=aarch64-unknown-linux-gnu-g++ AR=aarch64-unknown-linux-gnu-ar CFLAGS="-fpermissive" lib
    BUILD_BYPRODUCTS libtensorflow-lite.a ${CMAKE_BINARY_DIR}/tensorflow/src/tf/tensorflow/lite/tools/make/downloads/fft2d/fftsg.c
    INSTALL_COMMAND cp -f ${CMAKE_BINARY_DIR}/tensorflow/src/tf/tensorflow/lite/tools/make/gen/generic-aarch64_armv8-a/lib/libtensorflow-lite.a ${CMAKE_BINARY_DIR}/
)

if(NOT DOWNLOAD_HAPPENED)
  execute_process(
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
  COMMAND wget https://dl.google.com/coral/canned_models/mobilenet_v2_1.0_224_quant_edgetpu.tflite
  COMMAND wget http://download.tensorflow.org/models/tflite_11_05_08/mobilenet_v2_1.0_224_quant.tgz
  COMMAND tar xfz mobilenet_v2_1.0_224_quant.tgz
  COMMAND wget https://coral.withgoogle.com/static/docs/images/parrot.jpg
  )
  set(DOWNLOAD_HAPPENED TRUE CACHE BOOL "Has the download happened?" FORCE)
endif()

# Create Main project
add_executable(etdemo
	main.cpp
	utils.cc
	${CMAKE_BINARY_DIR}/tensorflow/src/tf/tensorflow/lite/tools/make/downloads/fft2d/fftsg.c
)

add_dependencies(etdemo edgetpu tf)

# For OpenCV
find_package(OpenCV REQUIRED)
if(OpenCV_FOUND)
	target_include_directories(etdemo PUBLIC ${OpenCV_INCLUDE_DIRS})
	target_link_libraries(etdemo ${OpenCV_LIBS})
endif()

target_link_libraries(etdemo ${CMAKE_BINARY_DIR}/libedgetpu.so)
target_link_libraries(etdemo ${CMAKE_BINARY_DIR}/libtensorflow-lite.a)

target_include_directories(etdemo PUBLIC ${CMAKE_BINARY_DIR}/edgetpu/src/edgetpu/libedgetpu/)
target_include_directories(etdemo PUBLIC ${CMAKE_BINARY_DIR}/tensorflow/src/tf)
target_include_directories(etdemo PUBLIC ${CMAKE_BINARY_DIR}/tensorflow/src/tf/tensorflow/lite/tools/make/downloads/flatbuffers/include)
# target_include_directories(etdemo PUBLIC ${CMAKE_SOURCE_DIR}/edgetpu/tensorflow/tensorflow)
# target_include_directories(etdemo PUBLIC ${CMAKE_SOURCE_DIR}/edgetpu/tensorflow/tensorflow/lite/tools/make/downloads)
# target_include_directories(etdemo PUBLIC ${CMAKE_SOURCE_DIR}/edgetpu/tensorflow/tensorflow/lite/tools/make/downloads/eigen )
# target_include_directories(etdemo PUBLIC ${CMAKE_SOURCE_DIR}/edgetpu/tensorflow/tensorflow/lite/tools/make/downloads/absl)
# target_include_directories(etdemo PUBLIC ${CMAKE_SOURCE_DIR}/edgetpu/tensorflow/tensorflow/lite/tools/make/downloads/gemmlowp)
# target_include_directories(etdemo PUBLIC ${CMAKE_SOURCE_DIR}/edgetpu/tensorflow/tensorlow/lite/tools/make/downloads/neon_2_sse)
# target_include_directories(etdemo PUBLIC ${CMAKE_SOURCE_DIR}/edgetpu/tensorflow/tensorflow/lite/tools/make/downloads/farmhash/src)

