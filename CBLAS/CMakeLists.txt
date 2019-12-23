message(STATUS "CBLAS enable")
enable_language(C)

set(LAPACK_INSTALL_EXPORT_NAME cblas-targets)
set(PKG_CONFIG_DIR ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
SET(LAPACK_BINARY_DIR ${CMAKE_BINARY_DIR})
include_directories(include ${LAPACK_BINARY_DIR}/include)
add_subdirectory(include)
add_subdirectory(src)

macro(append_subdir_files variable dirname)
get_directory_property(holder DIRECTORY ${dirname} DEFINITION ${variable})
foreach(depfile ${holder})
  list(APPEND ${variable} "${dirname}/${depfile}")
endforeach()
endmacro()

append_subdir_files(CBLAS_INCLUDE "include")
# install(FILES ${CBLAS_INCLUDE} ${LAPACK_BINARY_DIR}/include/cblas_mangling.h
#   DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
#   COMPONENT Development
#   )

# --------------------------------------------------
if(BUILD_TESTING)
  add_subdirectory(testing)
  add_subdirectory(examples)
endif()

if(NOT BLAS_FOUND)
  set(ALL_TARGETS ${ALL_TARGETS} blas)
endif()

# Export cblas targets from the
# install tree, if any.
set(_cblas_config_install_guard_target "")
if(ALL_TARGETS)
  install(TARGETS cblas
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/cblas-${LAPACK_VERSION}
    COMPONENT Development
    )
  # Choose one of the cblas targets to use as a guard for
  # cblas-config.cmake to load targets from the install tree.
  list(GET ALL_TARGETS 0 _cblas_config_install_guard_target)
endif()

# Export cblas targets from the build tree, if any.
set(_cblas_config_build_guard_target "")
if(ALL_TARGETS)
  export(TARGETS ${ALL_TARGETS} FILE cblas-targets.cmake)

  # Choose one of the cblas targets to use as a guard
  # for cblas-config.cmake to load targets from the build tree.
  list(GET ALL_TARGETS 0 _cblas_config_build_guard_target)
endif()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/cblas-config-version.cmake.in
  ${LAPACK_BINARY_DIR}/cblas-config-version.cmake @ONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/cblas-config-build.cmake.in
  ${LAPACK_BINARY_DIR}/cblas-config.cmake @ONLY)


configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cblas.pc.in ${CMAKE_CURRENT_BINARY_DIR}/cblas.pc @ONLY)
  install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/cblas.pc
  DESTINATION ${PKG_CONFIG_DIR}
  )

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/cblas-config-install.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/CMakeFiles/cblas-config.cmake @ONLY)
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/CMakeFiles/cblas-config.cmake
  ${LAPACK_BINARY_DIR}/cblas-config-version.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/cblas-${LAPACK_VERSION}
  )

#install(EXPORT cblas-targets
#  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/cblas-${LAPACK_VERSION}
#  COMPONENT Development
#  )
