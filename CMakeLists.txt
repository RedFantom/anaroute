cmake_minimum_required(VERSION 3.0)
#set(CMAKE_VERBOSE_MAKEFILE ON)
#set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(PROJECT_NAME "anaroute")
project(${PROJECT_NAME})

set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR}/bin)
set(PROJECT_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)

if (CMAKE_INSTALL_PREFIX)
  set(CMAKE_INSTALL_PREFIX_DIR ${CMAKE_INSTALL_PREFIX})
else()
  set(CMAKE_INSTALL_PREFIX_DIR ${CMAKE_CURRENT_SOURCE_DIR}/install)
endif()
message(STATUS "Install Prefix: ${CMAKE_INSTALL_PREFIX}")
unset(CMAKE_INSTALL_PREFIX)

# deal with GCC dual ABI issue 
if(NOT CMAKE_CXX_ABI)
    set(CMAKE_CXX_ABI 0 CACHE STRING
        "Choose the value for _GLIBCXX_USE_CXX11_ABI, options are: 0|1."
        FORCE)
endif(NOT CMAKE_CXX_ABI)
message(STATUS "CMAKE_CXX_ABI: _GLIBCXX_USE_CXX11_ABI=${CMAKE_CXX_ABI}")
add_definitions(-D_GLIBCXX_USE_CXX11_ABI=${CMAKE_CXX_ABI})

find_package(PkgConfig REQUIRED)

# Find LIMBO
set(LIMBO_ROOT_DIR ../limbo/limbo )
include_directories(${LIMBO_ROOT_DIR}/..)
message(STATUS "Find Limbo: ${LIMBO_ROOT_DIR}")
unset(LIMBO_DIR CACHE)

# Find Lemon
pkg_check_modules(LEMON REQUIRED lemon)
include_directories(${LEMON_INCLUDE_DIRS})
link_libraries(lemon)
message(STATUS "Found Lemon: " ${LEMON_FOUND})
message(STATUS "Lemon libraries: " ${LEMON_LIBRARIES})

# Find Boost
find_package(Boost 1.6 REQUIRED COMPONENTS system graph iostreams)

# Find Zlib
find_package(ZLIB REQUIRED)

# Find pybind11
find_package(pybind11 REQUIRED )
include_directories(${PYBIND11_INCLUDE_DIRS})
link_libraries(${PYBIND11_LIBS})
unset(PYBIND11_DIR CACHE)
set(PYBIND11_CPP_STANDARD -std=c++14)

# Find Libsparsehash
find_package(PkgConfig REQUIRED)
pkg_check_modules(SPARSE REQUIRED libsparsehash)
include_directories(${SPARSE_INCLUDE_DIRS})
link_libraries(${SPARSE_LIBRARIES})

# Python
pkg_check_modules(PYTHON REQUIRED python2=2.7)
include_directories(${PYTHON_INCLUDE_DIRS})
link_libraries(${PYTHON_LIBRARIES})
message(STATUS "Found Python2: " ${PYTHON_FOUND})

#Print out the used build type
IF(CMAKE_BUILD_TYPE MATCHES Debug)
    message(STATUS "Using build type DEBUG")
    #ENDIF(CMAKE_BUILD_TYPE MATCHES Debug)
ELSEIF(CMAKE_BUILD_TYPE MATCHES Release)
    message(STATUS "Using build type RELEASE")
    #ENDIF(CMAKE_BUILD_TYPE MATCHES Release)
ELSEIF(CMAKE_BUILD_TYPE MATCHES Profile)
    message(STATUS "Using build type PROFILE")
    #ENDIF(CMAKE_BUILD_TYPE MATCHES Profile)
ELSE()
    message(STATUS "Using build type DEFAULT: using Release flag")
    set(CMAKE_BUILD_TYPE Release)
ENDIF()

# CXX flags
set(CMAKE_CXX_FLAGS "-std=c++14 -Wall -fPIC")
set(CMAKE_CXX_FLAGS_DEBUG "-g -O0 -fno-inline ")
set(CMAKE_CXX_FLAGS_RELEASE "-O3")
set(CMAKE_CXX_FLAGS_PROFILE "-O3 -pg -Winline")

##################################################
#        Source Code                             #
##################################################
set(SRC
  src/util/util.cpp
  src/util/timeUsage.cpp
  src/util/MsgPrinter.cpp
  src/db/lef/layer/lefCutLayer.cpp
  src/db/lef/layer/lefRoutingLayer.cpp
  src/db/lef/lefSite.cpp
  src/db/lef/lefVia.cpp
  src/db/lef/lefViaRule.cpp
  src/db/lef/lefViaImpl.cpp
  src/db/dbLef.cpp
  src/db/dbTechfile.cpp
  src/db/dbPin.cpp
  src/db/dbCir.cpp
  src/db/dbNet.cpp
  src/parser/parser.cpp
  src/parser/parLef.cpp
  src/parser/parTech.cpp
  src/parser/parIspd08.cpp
  src/parser/parNetlist.cpp
  src/parser/parGds.cpp
  src/parser/parSymNet.cpp
  src/parser/parIOPin.cpp
  src/parser/parPower.cpp
  src/parser/parNetSpec.cpp
  src/gr/grMgr.cpp
  src/gr/grGridRoute.cpp
  src/gr/grAstar.cpp
  src/ta/taMgr.cpp
  src/acs/acsMgr.cpp
  src/dr/drMgr.cpp
  src/dr/drGridlessRoute.cpp
  src/dr/drGridRoute.cpp
  src/dr/drGridAstar.cpp
  src/dr/drAstar.cpp
  src/dr/drViaSelector.cpp
  src/dr/drRoutable.cpp
  src/dr/drSymmetry.cpp
  src/dr/drPower.cpp
  src/drc/drcMgr.cpp
  src/writer/writer.cpp
  src/writer/wrLayout.cpp
  src/writer/wrGrGuide.cpp
  src/writer/wrGds.cpp
  src/writer/wrDumb.cpp
  src/post/postMgr.cpp
)

set(SRC_EXE
  src/main/main.cpp
  src/main/anaroute.cpp
)

set(SRC_API
  src/api/apiPy.cpp  
)

add_executable(${PROJECT_NAME} ${SRC} ${SRC_EXE})
pybind11_add_module(${PROJECT_NAME}Py ${SRC} ${SRC_API})

##################################################
include_directories(${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${ZLIB_INCLUDE_DIR}
  ${Boost_INCLUDE_DIR}
  ${SPARSEHASH_ROOT_DIR}/include
  ${LIMBO_ROOT_DIR}/include
  ${PYBIND11_ROOT_DIR}/include
  ${LEMON_ROOT_DIR}/include
)

file(GLOB LIMBO_LIB
  #lefdef
  ${LIMBO_ROOT_DIR}/thirdparty/lefdef/5.8/def/libcdef.a
  ${LIMBO_ROOT_DIR}/thirdparty/lefdef/5.8/def/libcdefzlib.a
  ${LIMBO_ROOT_DIR}/thirdparty/lefdef/5.8/def/libdef.a
  ${LIMBO_ROOT_DIR}/thirdparty/lefdef/5.8/def/libdefzlib.a
  ${LIMBO_ROOT_DIR}/parsers/def/bison/libdefparser.a
  ${LIMBO_ROOT_DIR}/parsers/def/bison/libdefparseradapt.a
  ${LIMBO_ROOT_DIR}/parsers/lef/adapt/liblefparseradapt.a
  #limbo gdsparser
  ${LIMBO_ROOT_DIR}/parsers/gdsii/gdsdb/libgdsdb.a
  ${LIMBO_ROOT_DIR}/parsers/gdsii/stream/libgdsparser.a
  # limbo gzstream
  ${LIMBO_ROOT_DIR}/thirdparty/gzstream/libgzstream.a
)

target_link_libraries(${PROJECT_NAME} ${LIMBO_LIB} ${Boost_LIBRARIES} ${ZLIB_LIBRARIES})
target_link_libraries(${PROJECT_NAME}Py PUBLIC ${LIMBO_LIB} ${Boost_LIBRARIES} ${ZLIB_LIBRARIES})

#Install
install(TARGETS ${PROJECT_NAME} DESTINATION ${CMAKE_INSTALL_PREFIX_DIR}/bin)
