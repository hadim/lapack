message(STATUS "LAPACKE enable")
enable_language(C)

set(LAPACK_INSTALL_EXPORT_NAME lapacke-targets)

# Create a header file lapacke_mangling.h for the routines called in my C programs
include(FortranCInterface)
## Ensure that the fortran compiler and c compiler specified are compatible
FortranCInterface_VERIFY()
FortranCInterface_HEADER( ${LAPACK_BINARY_DIR}/include/lapacke_mangling.h
    MACRO_NAMESPACE "LAPACK_"
    SYMBOL_NAMESPACE "LAPACK_" )
if( NOT FortranCInterface_GLOBAL_FOUND OR NOT FortranCInterface_MODULE_FOUND)
  message(WARNING "Reverting to pre-defined include/lapacke_mangling.h")
  configure_file( include/lapacke_mangling_with_flags.h.in
                  ${LAPACK_BINARY_DIR}/include/lapacke_mangling.h )
endif ()

if (WIN32 AND NOT UNIX)
 add_definitions(-DHAVE_LAPACK_CONFIG_H -DLAPACK_COMPLEX_STRUCTURE)
 message (STATUS "Windows BUILD")
endif ()

get_directory_property( DirDefs COMPILE_DEFINITIONS )

include_directories( include ${LAPACK_BINARY_DIR}/include )
add_subdirectory(include)
add_subdirectory(src)
add_subdirectory(utils)

macro(append_subdir_files variable dirname)
get_directory_property(holder DIRECTORY ${dirname} DEFINITION ${variable})
foreach(depfile ${holder})
  list(APPEND ${variable} "${dirname}/${depfile}")
endforeach()
endmacro()

append_subdir_files(LAPACKE_INCLUDE "include")
append_subdir_files(SRC_OBJ "src")
append_subdir_files(SRCX_OBJ "src")
append_subdir_files(MATGEN_OBJ "src")
append_subdir_files(UTILS_OBJ "utils")

if (USE_XBLAS)
   add_library(lapacke ${SRC_OBJ} ${SRCX_OBJ} ${UTILS_OBJ})
   target_link_libraries(lapacke ${LAPACK_LIBRARIES} ${BLAS_LIBRARIES} ${XBLAS_LIBRARY})
else ()
  if (LAPACKE_WITH_TMG)
   add_library(lapacke ${SRC_OBJ} ${MATGEN_OBJ} ${UTILS_OBJ})
   target_link_libraries(lapacke tmglib ${LAPACK_LIBRARIES} ${BLAS_LIBRARIES})
  else ()
   add_library(lapacke ${SRC_OBJ} ${UTILS_OBJ})
   target_link_libraries(lapacke ${LAPACK_LIBRARIES} ${BLAS_LIBRARIES})
  endif()
endif()

lapack_install_library(lapacke)
install( FILES ${LAPACKE_INCLUDE} ${LAPACK_BINARY_DIR}/include/lapacke_mangling.h DESTINATION include )

if(BUILD_TESTING)
   add_subdirectory(example)
endif()


configure_file(${CMAKE_CURRENT_SOURCE_DIR}/lapacke.pc.in ${CMAKE_CURRENT_BINARY_DIR}/lapacke.pc @ONLY)
 install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/lapacke.pc
  DESTINATION ${PKG_CONFIG_DIR}
   )

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/lapacke-config-version.cmake.in
  ${LAPACK_BINARY_DIR}/lapacke-config-version.cmake @ONLY)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/lapacke-config-build.cmake.in
  ${LAPACK_BINARY_DIR}/lapacke-config.cmake @ONLY)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/lapacke-config-install.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/CMakeFiles/lapacke-config.cmake @ONLY)
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/CMakeFiles/lapacke-config.cmake
  ${LAPACK_BINARY_DIR}/lapacke-config-version.cmake
  DESTINATION ${LIBRARY_DIR}/cmake/lapacke-${LAPACK_VERSION}
  )

install(EXPORT lapacke-targets
  DESTINATION ${LIBRARY_DIR}/cmake/lapacke-${LAPACK_VERSION})
