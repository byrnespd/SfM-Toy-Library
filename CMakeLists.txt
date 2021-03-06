#  The MIT License (MIT)
# 
#  Copyright (c) 2013 Roy Shilkrot
#   
#  Permission is hereby granted, free of charge, to any person obtaining a copy
#  of this software and associated documentation files (the "Software"), to deal
#  in the Software without restriction, including without limitation the rights
#  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
#  copies of the Software, and to permit persons to whom the Software is
#  furnished to do so, subject to the following conditions:
#   
#  The above copyright notice and this permission notice shall be included in all
#  copies or substantial portions of the Software.
#   
#  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
#  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
#  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
#  SOFTWARE.

project(SfMToyExample)
cmake_minimum_required(VERSION 2.8)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")

find_package(OpenCV REQUIRED)
find_package(OpenMP)
find_package(OpenGL)

add_subdirectory(SfMToyLib)

############### Find Eigen ###############
find_package(Eigen3 REQUIRED)
include_directories(${EIGEN3_INCLUDE_DIR})
###########################################

SET(USE_GUI TRUE CACHE BOOL "Build with GUI or just CLI")

SET(USE_QT TRUE CACHE BOOL "Use QT for GUI")

IF(USE_GUI)
	SET(USE_QT TRUE) #Force Qt if using GUI
	ADD_DEFINITIONS("-DHAVE_GUI")
ENDIF(USE_GUI)

IF(USE_QT)
	########## Qt stuff ##########
	find_package(Qt4 REQUIRED QtGui QtOpenGL QtXml)
	
	ADD_DEFINITIONS(${QT_DEFINITIONS})
	include_directories(${QT_INCLUDES})
	include(${QT_USE_FILE}) 
	
	QT4_WRAP_UI( MY_UI_HDRS sfmtoy.ui )
	QT4_WRAP_CPP( MY_MOC_SRCS ViewerInterface.h sfmviewer.h )
	SOURCE_GROUP(QT_MOC_SRCS FILES ${MY_MOC_SRCS})
	
	INCLUDE_DIRECTORIES( ${CMAKE_BINARY_DIR} ) #qt wrapping creates some sources
	######### End Qt Stuff #########
	
	############### Find LibQGLViewer ###############
	set(QGLVIEWER_DIR_HINT "" CACHE PATH "libQGLViewer directory")
	FIND_LIBRARY(QGLVIEWER_LIB QGLViewer2 PATHS ${QGLVIEWER_DIR_HINT} "${QGLVIEWER_DIR_HINT}/QGLViewer/release")
	IF(NOT QGLVIEWER_LIB)
	    FIND_LIBRARY(QGLVIEWER_LIB QGLViewer PATHS ${QGLVIEWER_DIR_HINT} "${QGLVIEWER_DIR_HINT}/QGLViewer/release")
	ENDIF()
	IF(NOT QGLVIEWER_LIB)
	   message(FATAL_ERROR "QGLVIEWER not found")
	ENDIF(NOT QGLVIEWER_LIB)
	find_path(QGLVIEWER_INCLUDE "QGLViewer/qglviewer.h" PATHS ${QGLVIEWER_DIR_HINT})
	if(NOT QGLVIEWER_INCLUDE)
		message(FATAL_ERROR "QGLViewer includes not found")
	ENDIF(NOT QGLVIEWER_INCLUDE)
	include_directories(${QGLVIEWER_INCLUDE})
	############################################
ENDIF(USE_QT)

IF(APPLE)
#	set( COCOA_LIBS ${CMAKE_OSX_SYSROOT}/System/Library/Frameworks/Cocoa.framework )
	set( CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -F/usr/local/lib -L/opt/local/lib")
	INCLUDE_DIRECTORIES ( /System/Library/Frameworks )
	FIND_LIBRARY(COCOA_LIBRARY Cocoa)
	FIND_LIBRARY(GLUT_LIBRARY GLUT )
	SET(OpenGL_LIBS ${COCOA_LIBRARY} ${GLUT_LIBRARY} ${OPENGL_gl_LIBRARY} ${OPENGL_glu_LIBRARY})
ENDIF(APPLE)

set( CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -D__SFM__DEBUG__" )

if(OPENMP_FOUND)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS} -DHAVE_OPENMP")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS} -DHAVE_OPENMP")
	set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()

include_directories(
	SfMToyLib
	3rdparty/SSBA-3.0
	)

# GUI part
add_executable(SfMToyUI 
	ViewerInterface.h ViewerInterface.cpp
	sfmviewer.h sfmviewer.cpp
	${MY_UI_HDRS}
	${MY_MOC_SRCS}
	main.cpp
	)

target_link_libraries(SfMToyUI 
	SfMToyLibrary
	${OpenCV_LIBS}
	${QT_LIBRARIES} 
	${QGLVIEWER_LIB}
	${OPENGL_LIBRARIES}
	)

