cmake_minimum_required(VERSION 3.0)
project(libjpeg-cmake VERSION 1.0)

configure_file(CMakeLists-IJG.txt ${PROJECT_SOURCE_DIR}/src/CMakeLists.txt COPYONLY)
configure_file(jconfig.h.in ${PROJECT_SOURCE_DIR}/src/jconfig.h.in COPYONLY)
add_subdirectory(src)
