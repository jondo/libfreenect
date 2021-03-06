######################################################################################
# Build for main library
######################################################################################

include_directories (${CMAKE_CURRENT_SOURCE_DIR})

set(CMAKE_C_FLAGS "-Wall")

include_directories(${LIBUSB_1_INCLUDE_DIRS})
IF(WIN32)
  LIST(APPEND SRC core.c tilt.c cameras.c usb_libusb10.c ../platform/windows/libusb10emu/libusb-1.0/libusbemu.cpp ../platform/windows/libusb10emu/libusb-1.0/failguard.cpp)
  set_source_files_properties(${SRC} PROPERTIES LANGUAGE CXX)
ELSE(WIN32)
  LIST(APPEND SRC core.c tilt.c cameras.c usb_libusb10.c)
ENDIF(WIN32)

add_library (freenect SHARED ${SRC})
set_target_properties ( freenect PROPERTIES
  VERSION ${PROJECT_VER}
  SOVERSION ${PROJECT_APIVER})	
install (TARGETS freenect
  DESTINATION "${PROJECT_LIBRARY_INSTALL_DIR}")

add_library (freenectstatic STATIC ${SRC})
set_target_properties (freenectstatic PROPERTIES OUTPUT_NAME freenect)
IF(UNIX AND NOT APPLE)
  SET_TARGET_PROPERTIES (freenectstatic PROPERTIES COMPILE_FLAGS "-fPIC")
ENDIF()
install (TARGETS freenectstatic
  DESTINATION "${PROJECT_LIBRARY_INSTALL_DIR}")

target_link_libraries (freenect ${LIBUSB_1_LIBRARIES})
target_link_libraries (freenectstatic ${LIBUSB_1_LIBRARIES})

# Install the header file
install (FILES "../include/libfreenect.h"
  DESTINATION ${PROJECT_INCLUDE_INSTALL_DIR})

IF(UNIX AND NOT APPLE)
  # Produce a pkg-config file for linking against the shared lib
  configure_file ("libfreenect.pc.in" "libfreenect.pc" @ONLY)
  install (FILES "${CMAKE_CURRENT_BINARY_DIR}/libfreenect.pc"
    DESTINATION "${PROJECT_LIBRARY_INSTALL_DIR}/pkgconfig")
ENDIF()
