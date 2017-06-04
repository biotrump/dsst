cmake_minimum_required(VERSION 3.0)
project(DSST_ferz)

#find_package( FFTW REQUIRED)
find_library(FFTW_LIBRARY
     NAMES fftw3f fftw3 fftw3_omp fftw)

message("FFTW_LIBRARY=${FFTW_LIBRARY}")
#find_package(OpenCV REQUIRED)
# --------------------------------------------------
message("OPENCV_BRANCH=$ENV{OPENCV_BRANCH}")
string( SUBSTRING "$ENV{OPENCV_BRANCH}" 0 4 OPENCV_MAJOR)
message("OPENCV_MAJOR=${OPENCV_MAJOR}")

if(ANDROID_NDK)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIE -pie")
	# if(${APP_ABI} STREQUAL "armeabi-v7a" )
		#openCV 3.2.0 SDK by "make install"
		#CV_MAJOR_VERSION==3
		set(OpenCV_DIR $ENV{OPENCV_OUT}/${APP_ABI}/sdk/native/jni)
		include( "${OpenCV_DIR}/OpenCVConfig.cmake" )
		message("++++OpenCV_DIR:${OpenCV_DIR}")
		FIND_PACKAGE( OpenCV REQUIRED)
		if(OpenCV_FOUND)
			message("found")
			message( "OpenCV_INCLUDE_DIRS:${OpenCV_INCLUDE_DIRS}" )
			message( "OpenCV_INCLUDES:${OpenCV_INCLUDES}")
			message( "OpenCV_LIBRARIES:${OpenCV_LIBRARIES}")
			message( "OpenCV_LIBS:${OpenCV_LIBS}")
			message( "OpenCV_DEFINITIONS:${OpenCV_DEFINITIONS}")
			add_definitions(-DUSE_OPENCV=${USE_OPENCV})
			set(OPENCV_BRANCH $ENV{OPENCV_BRANCH})
			include_directories(${OpenCV_INCLUDE_DIRS})
		else()
			message(FATAL_ERROR "openCV not foud")
		endif()
	#endif()
elseif(ARM_LINUX_SYSROOT)
else()
	# Tell CMake to run moc when necessary:
	set(CMAKE_AUTOMOC ON)
	# As moc files are generated in the binary dir, tell CMake
	# to always look for includes there:
	set(CMAKE_INCLUDE_CURRENT_DIR ON)

	# Widgets finds its own dependencies (QtGui and QtCore).
	#find_package(Qt5Widgets)
	#  find_package(Qt5Declarative)

	# The Qt5Widgets_INCLUDES also includes the include directories for
	# dependencies QtCore and QtGui
	#include_directories(${Qt5Widgets_INCLUDES})

	# We need add -DQT_WIDGETS_LIB when using QtWidgets in Qt 5.
	#add_definitions(${Qt5Widgets_DEFINITIONS})
	#opencv :: cmake/OpenCVFindLibsGUI.cmake
	find_package(Qt5Core)
	find_package(Qt5Gui)
	find_package(Qt5Widgets)
	find_package(Qt5Test)
	find_package(Qt5Concurrent)
	message("Qt5Core_FOUND=${Qt5Core_FOUND}")
	message("Qt5Gui_FOUND=${Qt5Gui_FOUND}")
	message("Qt5Widgets_FOUND=${Qt5Widgets_FOUND}")
	message("Qt5Test_FOUND=${Qt5Test_FOUND}")
	message("Qt5Concurrent_FOUND=${Qt5Concurrent_FOUND}")
	if(Qt5Core_FOUND AND Qt5Gui_FOUND AND Qt5Widgets_FOUND AND Qt5Test_FOUND AND Qt5Concurrent_FOUND)
	set(HAVE_QT5 ON)
	set(HAVE_QT  ON)
	find_package(Qt5OpenGL)
	if(Qt5OpenGL_FOUND)
	    set(QT_QTOPENGL_FOUND ON)
	endif()
	endif()
	# Executables fail to build with Qt 5 in the default configuration
	# without -fPIE. We add that here.
	#  set(CMAKE_CXX_FLAGS "${Qt5Widgets_EXECUTABLE_COMPILE_FLAGS}")

	# --------------------------------------------------
	#opencv should be built first!
	set(OpenCV_DIR $ENV{OPENCV_OUT})
	#set(OpenCV_DIR "/home/thomas/opencv/build_2.4.13/debug")
	#set(OpenCV_DIR "/home/thomas/build/bcv-x86.gl/out/opencv/2.4.x-static/x86_64")
	#set(OpenCV_DIR "/home/thomas/opencv/build_3.2.0/debug")
	#add_definitions(-DOPENCV_VER=2)
	message("++++OpenCV_DIR:${OpenCV_DIR}")
	FIND_PACKAGE( OpenCV REQUIRED)
	if(OpenCV_FOUND)
		message("found")
		message( "OpenCV_INCLUDE_DIRS:${OpenCV_INCLUDE_DIRS}" )
		message( "OpenCV_INCLUDES:${OpenCV_INCLUDES}")
		message( "OpenCV_LIBRARIES:${OpenCV_LIBRARIES}")
		message( "OpenCV_LIBS:${OpenCV_LIBS}")
		message( "OpenCV_DEFINITIONS:${OpenCV_DEFINITIONS}")
		#set(OPENCV_BRANCH $ENV{OPENCV_BRANCH})
		add_definitions(-DUSE_OPENCV=${USE_OPENCV})
	else()
		message(FATAL_ERROR "openCV not foud")
	endif()

	message("*** OpenCV_LIBS:${OpenCV_LIBS}")
	message("*** OpenCV_LIBRARIES:${OpenCV_LIBRARIES}")

endif(ANDROID_NDK)
# --------------------------------------------------

if(NOT WIN32)
	ADD_DEFINITIONS("-std=c++11 -O3")
endif(NOT WIN32)

include_directories(dsst) 
FILE(GLOB_RECURSE sourcefiles "dsst/*.cpp")
add_executable( dsst ${sourcefiles} )
target_link_libraries( dsst 
			${OpenCV_LIBS}
			${FFTW_LIBRARY})

SET(CMAKE_BUILD_TYPE "Debug")
IF(CMAKE_COMPILER_IS_GNUCC)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0")
    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -O0")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -Wall -O3")
ENDIF(CMAKE_COMPILER_IS_GNUCC)


