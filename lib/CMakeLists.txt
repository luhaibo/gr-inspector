# Copyright 2011,2012 Free Software Foundation, Inc.
#
# This file is part of GNU Radio
#
# GNU Radio is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 3, or (at your option)
# any later version.
#
# GNU Radio is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with GNU Radio; see the file COPYING.  If not, write to
# the Free Software Foundation, Inc., 51 Franklin Street,
# Boston, MA 02110-1301, USA.

#set (CMAKE_CXX_FLAGS "-lqwtplot3d -lGLU") 
########################################################################
# QT GUI file generation stuff (Q_OBJECT)
########################################################################
set(QTGUI_MOC_HEADERS
		inspector_form.h vis3d_vf_form.h)
QT4_WRAP_CPP(QTGUI_MOC_OUTFILES ${QTGUI_MOC_HEADERS})

########################################################################
# Setup library
########################################################################
include(GrPlatform) #define LIB_SUFFIX

include_directories(${Boost_INCLUDE_DIR} ${QT_INCLUDE_DIRS})
link_directories(${Boost_LIBRARY_DIRS} ${GNURADIO_ALL_LIBRARIES})
list(APPEND inspector_sources
		${QTGUI_MOC_OUTFILES}
    signal_separator_c_impl.cc
    signal_detector_cvf_impl.cc
    signal_extractor_c_impl.cc
    qtgui_inspector_sink_vf_impl.cc
		     vis3d_vf_form.cc
		inspector_form.cc
		signal_marker.cc
		ofdm_bouzegzi_c_impl.cc
    ofdm_zkf_c_impl.cc
    ofdm_synchronizer_cc_impl.cc
    vis3d_vf_impl.cc
    )

include(GrPython)
if(ENABLE_PYTHON)
	add_definitions(-DENABLE_PYTHON)
	list(APPEND qtgui_libs
			${PYTHON_LIBRARIES}
			)
endif(ENABLE_PYTHON)

set(inspector_sources "${inspector_sources}" PARENT_SCOPE)
if(NOT inspector_sources)
	MESSAGE(STATUS "No C++ sources... skipping lib/")
	return()
endif(NOT inspector_sources)


add_library(gnuradio-inspector SHARED ${inspector_sources})
target_link_libraries(gnuradio-inspector ${Boost_LIBRARIES} ${QWT_LIBRARIES} ${PYTHON_LIBRARIES} ${QWTPLOT3D_LIBRARIES} ${QT_LIBRARIES} ${OPENGL_LIBRARIES} ${GNURADIO_ALL_LIBRARIES})
set_target_properties(gnuradio-inspector PROPERTIES DEFINE_SYMBOL "gnuradio_inspector_EXPORTS")

if(APPLE)
    set_target_properties(gnuradio-inspector PROPERTIES
        INSTALL_NAME_DIR "${CMAKE_INSTALL_PREFIX}/lib"
    )
endif(APPLE)

########################################################################
# Install built library files
########################################################################
install(TARGETS gnuradio-inspector
    LIBRARY DESTINATION lib${LIB_SUFFIX} # .so/.dylib file
    ARCHIVE DESTINATION lib${LIB_SUFFIX} # .lib file
    RUNTIME DESTINATION bin              # .dll file
)

########################################################################
# Build and register unit test
########################################################################
include(GrTest)

include_directories(${CPPUNIT_INCLUDE_DIRS})

list(APPEND test_inspector_sources
    ${CMAKE_CURRENT_SOURCE_DIR}/test_inspector.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/qa_inspector.cc
)

add_executable(test-inspector ${test_inspector_sources})

target_link_libraries(
  test-inspector
  ${GNURADIO_RUNTIME_LIBRARIES}
  ${Boost_LIBRARIES}
  ${CPPUNIT_LIBRARIES}
  ${QWT_LIBRARIES}
  ${QT_LIBRARIES}
  ${QWTPLOT3D_LIBRARIES}
  gnuradio-inspector
)

GR_ADD_TEST(test_inspector test-inspector)

########################################################################
# Print summary
########################################################################
message(STATUS "Using install prefix: ${CMAKE_INSTALL_PREFIX}")
message(STATUS "Building for version: ${VERSION} / ${LIBVER}")
