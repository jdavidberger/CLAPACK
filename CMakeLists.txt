cmake_minimum_required(VERSION 2.6)
project(CLAPACK C)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

if(WIN32 AND NOT CYGWIN)
  set(SECOND_SRC  ${CLAPACK_SOURCE_DIR}/INSTALL/winsecond.c)
  set(DSECOND_SRC  ${CLAPACK_SOURCE_DIR}/INSTALL/windsecnd.c)
  add_definitions(-DNO_ISATTY -DMSDOS -DUSE_CLOCK)
else()
  set(SECOND_SRC  ${CLAPACK_SOURCE_DIR}/INSTALL/second.c)
  set(DSECOND_SRC  ${CLAPACK_SOURCE_DIR}/INSTALL/dsecnd.c)
endif()

option(USE_BLAS_WRAP "pre-pend f2c_ to each function in blas" OFF)
if(NOT USE_BLAS_WRAP)
# _zrotg_ seems to be missing in the wrap header
  add_definitions(-DNO_BLAS_WRAP)
endif()
include_directories(${CLAPACK_SOURCE_DIR}/INCLUDE)
add_subdirectory(F2CLIBS)
include_directories(${CLAPACK_SOURCE_DIR}/F2CLIBS/libf2c)
add_subdirectory(BLAS)
add_subdirectory(SRC)
add_subdirectory(LAPACKE)
add_subdirectory(CBLAS)

if( BUILD_TESTING )
  enable_testing()
  include(CTest)
  add_subdirectory(TESTING)
endif()

set(CLAPACK_VERSION 3.2.1)
set(CPACK_PACKAGE_VERSION_MAJOR 3)
set(CPACK_PACKAGE_VERSION_MINOR 2)
set(CPACK_PACKAGE_VERSION_PATCH 1)
include(CPack)
export(TARGETS f2c blas lapack FILE clapack-targets.cmake)
configure_file(${CLAPACK_SOURCE_DIR}/clapack-config-version.cmake.in
  ${CLAPACK_BINARY_DIR}/clapack-config-version.cmake @ONLY)
configure_file(${CLAPACK_SOURCE_DIR}/clapack-config.cmake.in
  ${CLAPACK_BINARY_DIR}/clapack-config.cmake @ONLY)

install(DIRECTORY LAPACKE/include/ INCLUDE/ CBLAS/include/ DESTINATION include FILES_MATCHING PATTERN "*.h")
install(FILES F2CLIBS/libf2c/f2c.h DESTINATION include)
