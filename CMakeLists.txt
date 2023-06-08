# I don't really know what the minimal cmake version is.
cmake_minimum_required (VERSION 3.9)

project(consumecasadi)

file(MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/casadiLocation")
set(CMAKE_INSTALL_PREFIX_BACKUP "${CMAKE_INSTALL_PREFIX}")


set(casadi_library_name ${CMAKE_SHARED_LIBRARY_PREFIX}casadi${CMAKE_SHARED_LIBRARY_SUFFIX})
set(casadi_library_file ${CMAKE_CURRENT_BINARY_DIR}/casadiLocation/lib/${casadi_library_name})

include(ProcessorCount)
ProcessorCount(NumberOfProcessors)

execute_process(
  COMMAND ${CMAKE_COMMAND} -S . -B build -G "Unix Makefiles" -DCMAKE_CXX_COMPILER_LAUNCHER=${CMAKE_CXX_COMPILER_LAUNCHER} -DWITH_HIGHS=ON -DWITH_SUNDIALS=ON -DWITH_IPOPT=ON -DWITH_BUILD_SUNDIALS=ON -DWITH_BUILD_IPOPT=ON -DWITH_BUILD_HIGHS=ON -DWITH_BUILD_MUMPS=ON -DWITH_BUILD_METIS=ON -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/casadiLocation
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/casadi
)


add_custom_command(OUTPUT  ${casadi_library_file}
  COMMAND ${CMAKE_COMMAND} --build build -j${NumberOfProcessors} --target install
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/casadi
    USES_TERMINAL)
  
  # create a target out of the ipopt compilation that depends on the generated library:
add_custom_target(casadi_build_target ALL
  DEPENDS ${casadi_library_file}
)

set(casadi_include_dir "${CMAKE_CURRENT_BINARY_DIR}/casadiLocation/include")

add_library(casadi INTERFACE)
add_dependencies(casadi casadi_build_target ${casadi_library_file})
target_link_libraries(casadi INTERFACE ${casadi_library_file})
target_include_directories(casadi SYSTEM INTERFACE ${casadi_include_dir})
