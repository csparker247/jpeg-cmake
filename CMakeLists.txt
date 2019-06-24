cmake_minimum_required(VERSION 3.0)
project(jpeg-cmake VERSION 1.0)

# Use configure_file insted of file(COPY)
# to ensure files are updated in src directory
configure_file(
  resources/CMakeLists.txt
  ${PROJECT_SOURCE_DIR}/libjpeg/CMakeLists.txt
  COPYONLY
)
configure_file(
  resources/jconfig.h.in
  ${PROJECT_SOURCE_DIR}/libjpeg/jconfig.h.in
  COPYONLY
)
configure_file(
  resources/libjpeg.pc.cmakein
  ${PROJECT_SOURCE_DIR}/libjpeg/libjpeg.pc.cmakein
  COPYONLY
)
configure_file(
  resources/libjpeg.la.in
  ${PROJECT_SOURCE_DIR}/libjpeg/libjpeg.la.in
  COPYONLY
)

# Build libjpeg
add_subdirectory(libjpeg)
