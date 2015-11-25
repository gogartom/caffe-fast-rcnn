cmake_minimum_required(VERSION 2.8.7)

# ---[ Caffe project
project(Caffe C CXX)

# ---[ Using cmake scripts and modules
list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake/Modules)

include(cmake/Utils.cmake)
include(cmake/Targets.cmake)
include(cmake/Misc.cmake)
include(cmake/Summary.cmake)
include(cmake/ConfigGen.cmake)

set(CUDA_TOOLKIT_ROOT_DIR /storage/plzen1/home/hudler/cuda-7.0/)
set(GFLAGS_INCLUDE_DIR /storage/plzen1/home/gogartom/gflags/installation/include)
set(GFLAGS_LIBRARY /storage/plzen1/home/gogartom/gflags/installation/lib/libgflags.a)
set(GLOG_INCLUDE_DIR /storage/plzen1/home/gogartom/glog-0.3.3/installation/include)
set(GLOG_LIBRARY /storage/plzen1/home/gogartom/glog-0.3.3/installation/lib/libglog.so)

set(PROTOBUF_INCLUDE_DIR /storage/plzen1/home/gogartom/protobuf-2.6/installation/include)
set(PROTOBUF_LIBRARY /storage/plzen1/home/gogartom/protobuf-2.6/installation/lib/libprotobuf.so)
set(PROTOBUF_PROTOC_EXECUTABLE /storage/plzen1/home/gogartom/protobuf-2.6/installation/bin/protoc)

set(LMDB_INCLUDE_DIR /storage/plzen1/home/gogartom/lmdb/libraries/liblmdb/installation/include)
set(LMDB_LIBRARIES /storage/plzen1/home/gogartom/lmdb/libraries/liblmdb/installation/lib/liblmdb.so)
set(LevelDB_INCLUDE /storage/plzen1/home/gogartom/leveldb/include)
set(LevelDB_LIBRARY /storage/plzen1/home/gogartom/leveldb/libleveldb.so)

set(Snappy_LIBRARIES /storage/plzen1/home/gogartom/snappy/installation/lib/libsnappy.so)
set(Snappy_INCLUDE_DIR /storage/plzen1/home/gogartom/snappy/installation/include)

set(Atlas_CBLAS_INCLUDE_DIR /afs/.ics.muni.cz/software/atlas-3.10.1/gcc4.7.0/include)
set(Atlas_CBLAS_LIBRARY /afs/.ics.muni.cz/software/atlas-3.10.1/gcc4.7.0/lib/libcblas.a)
set(Atlas_CLAPACK_INCLUDE_DIR /afs/.ics.muni.cz/software/atlas-3.10.1/gcc4.7.0/include)
set(Atlas_LAPACK_LIBRARY /afs/.ics.muni.cz/software/atlas-3.10.1/gcc4.7.0/lib/liblapack.a)
set(Atlas_BLAS_LIBRARY /afs/.ics.muni.cz/software/atlas-3.10.1/gcc4.7.0/lib/libatlas.a)

set(PYTHON_LIBRARY /software/python-2.7.6/gcc/lib/libpython2.7.so)
set(PYTHON_INCLUDE_DIR /software/python-2.7.6/gcc/include/python2.7)
set(Boost_INCLUDE_DIR /storage/plzen1/home/gogartom/boost_py_2.7.6/installation/include)


# ---[ Options
caffe_option(CPU_ONLY  "Build Caffe wihtout CUDA support" OFF) # TODO: rename to USE_CUDA
caffe_option(USE_CUDNN "Build Caffe with cuDNN libary support" ON IF NOT CPU_ONLY)
caffe_option(BUILD_SHARED_LIBS "Build shared libraries" ON)
caffe_option(BUILD_python "Build Python wrapper" ON)
set(python_version "2" CACHE STRING "Specify which python version to use")
caffe_option(BUILD_matlab "Build Matlab wrapper" OFF IF UNIX OR APPLE)
caffe_option(BUILD_docs   "Build documentation" ON IF UNIX OR APPLE)
caffe_option(BUILD_python_layer "Build the caffe python layer" ON)

# ---[ Dependencies
include(cmake/Dependencies.cmake)

# ---[ Flags
if(UNIX OR APPLE)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -Wall")
endif()

if(USE_libstdcpp)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libstdc++")
  message("-- Warning: forcing libstdc++ (controlled by USE_libstdcpp option in cmake)")
endif()

add_definitions(-DGTEST_USE_OWN_TR1_TUPLE)

# ---[ Warnings
caffe_warnings_disable(CMAKE_CXX_FLAGS -Wno-sign-compare -Wno-uninitialized)

# ---[ Config generation
configure_file(cmake/Templates/caffe_config.h.in "${PROJECT_BINARY_DIR}/caffe_config.h")

# ---[ Includes
set(Caffe_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include)
include_directories(${Caffe_INCLUDE_DIR} ${PROJECT_BINARY_DIR})
include_directories(BEFORE src) # This is needed for gtest.

# ---[ Subdirectories
add_subdirectory(src/gtest)
add_subdirectory(src/caffe)
add_subdirectory(tools)
add_subdirectory(examples)
add_subdirectory(python)
add_subdirectory(matlab)
add_subdirectory(docs)

# ---[ Linter target
add_custom_target(lint COMMAND ${CMAKE_COMMAND} -P ${PROJECT_SOURCE_DIR}/cmake/lint.cmake)

# ---[ Configuration summary
caffe_print_configuration_summary()

# ---[ Export configs generation
caffe_generate_export_configs()
