cmake_minimum_required(VERSION 2.8)
project( denseFlow )

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fPIC")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()


find_package( OpenCV REQUIRED )
find_package( LibZip REQUIRED )

# BOOST
FIND_PACKAGE(PythonLibs REQUIRED)

if (PYTHONINTERP_FOUND)
  if (UNIX AND NOT APPLE)
    if (PYTHON_VERSION_MAJOR EQUAL 3)
        FIND_PACKAGE(Boost COMPONENTS python${PYTHON_VERSION_SUFFIX})
        FIND_PACKAGE(PythonInterp 3)
        FIND_PACKAGE(PythonLibs 3 REQUIRED)
    else()
        FIND_PACKAGE(Boost COMPONENTS python)
        FIND_PACKAGE(PythonInterp)
        FIND_PACKAGE(PythonLibs REQUIRED)
    endif()
  else()	
    if (PYTHON_VERSION_MAJOR EQUAL 3)
        FIND_PACKAGE(Boost COMPONENTS python${PYTHON_VERSION_MAJOR}${PYTHON_VERSION_MINOR})
        FIND_PACKAGE(PythonInterp 3)
        FIND_PACKAGE(PythonLibs 3 REQUIRED)
    else()
        FIND_PACKAGE(Boost COMPONENTS python${PYTHON_VERSION_MAJOR}${PYTHON_VERSION_MINOR})
        FIND_PACKAGE(PythonInterp)
        FIND_PACKAGE(PythonLibs REQUIRED)
    endif()
  endif()
else()
    message("Python not found")
endif()

if(LIBZIP_VERSION VERSION_LESS 0.11)
    #old version LibZip
    add_definitions(-DUSE_OBSEL_LIBZIP)
endif()

include_directories( ${OpenCV_INCLUDE_DIRS} ${LIBZIP_INCLUDE_DIR_ZIP} ${LIBZIP_INCLUDE_DIR_ZIPCONF} include/ include/easylogging++/src include/CLUE/include)
include_directories(SYSTEM ${Boost_INCLUDE_DIR} ${PYTHON_INCLUDE_DIR})

add_library(denseflow src/common.cpp src/dense_flow.cpp src/dense_flow_gpu.cpp src/dense_warp_flow_gpu.cpp src/zip_utils.cpp)
target_link_libraries( denseflow ${OpenCV_LIBS} ${LIBZIP_LIBRARY})

add_library(pydenseflow SHARED src/py_denseflow.cpp)
target_link_libraries(pydenseflow
        denseflow
        ${Boost_LIBRARIES} ${PYTHON_LIBRARIES} ${OpenCV_LIBS}
        )

add_executable( extract_cpu tools/extract_flow.cpp)
target_link_libraries( extract_cpu ${OpenCV_LIBS} ${LIBZIP_LIBRARY} denseflow)

add_executable( extract_gpu tools/extract_flow_gpu.cpp)
target_link_libraries( extract_gpu ${OpenCV_LIBS} ${LIBZIP_LIBRARY} denseflow)

add_executable( extract_warp_gpu tools/extract_warp_flow_gpu.cpp)
target_link_libraries( extract_warp_gpu ${OpenCV_LIBS} ${LIBZIP_LIBRARY} denseflow)
