cmake_minimum_required (VERSION 3.9)
# 3.9 is needed for CheckIPOSupported and for doxygen_add_docs.

project(consumecasadi LANGUAGES C CXX)


#for multiple compiler compatibility:
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

#for development:
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

file(MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/casadiLocation")
set(CMAKE_INSTALL_PREFIX_BACKUP "${CMAKE_INSTALL_PREFIX}")


set(casadi_library_name ${CMAKE_SHARED_LIBRARY_PREFIX}casadi${CMAKE_SHARED_LIBRARY_SUFFIX})
set(casadi_library_file ${CMAKE_CURRENT_BINARY_DIR}/casadiLocation/lib/${casadi_library_name})

include(ProcessorCount)
ProcessorCount(NumberOfProcessors)

add_custom_command(OUTPUT  ${casadi_library_file}
  COMMAND ${CMAKE_COMMAND} -S . -B build -DCMAKE_CXX_COMPILER_LAUNCHER=${CMAKE_CXX_COMPILER_LAUNCHER} -DWITH_HIGHS=ON -DWITH_IPOPT=ON -DWITH_BUILD_IPOPT=ON -DWITH_BUILD_HIGHS=ON -DWITH_BUILD_MUMPS=ON -DWITH_BUILD_METIS=ON -DCMAKE_INSTALL_PREFIX="${CMAKE_CURRENT_BINARY_DIR}/casadiLocation" && cmake --build build -j${NumberOfProcessors} --target install
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

add_subdirectory(src)

add_subdirectory(docs)
add_subdirectory(tests)

