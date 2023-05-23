cmake_minimum_required (VERSION 3.9)
# 3.9 is needed for CheckIPOSupported and for doxygen_add_docs.

project(consumecasadi LANGUAGES C CXX)

include(CheckIPOSupported)

#for multiple compiler compatibility:
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

#for development:
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)


# for debug builds with gcc use the debug configuration of the standard libraries:

# We don't do that with Clang, although there is _LIBCPP_DEBUG, because
# this fails with some part of our json dependencies.
# Also MacOS doesn't ship the debug versions of standard types,
# therefore we would forego the debug macro for Apple's Clang.
add_compile_definitions("$<$<CONFIG:DEBUG>:_GLIBCXX_DEBUG>")

option(BUILD_WITH_LIBCPP "Build with libc++ instead of libstdc++, if the compiler is clang." OFF)
# include the googletest project for tests:
# here also the standard library has to be set, if compiling with llvms libc++.
# Our own compile options are set later.
if (BUILD_WITH_LIBCPP AND CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options("-stdlib=libc++")
  add_link_options("-stdlib=libc++")
  message(STATUS "Will compile with libc++.")
endif()
enable_testing()

add_subdirectory(googletest EXCLUDE_FROM_ALL)


# An option to exclude IPO from the release type to make better use of a compiler cache (like ccache) for development:
if (NOT DEVEL_RELEASE)

  # Funny enough, this is actually detrimental on my machine.
  # add_compile_options("$<$<CONFIG:RELEASE>:-march=native>")

  # for Release types add link-time optimization
  check_ipo_supported(RESULT is_ipo_supported OUTPUT lto_error)
  if(is_ipo_supported)
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION_DEBUG OFF)
    message(STATUS "Interprocedural optimization supported!")
  else()
    message(STATUS "NO INTERPROCESS OPTIMIZATION AVAILABLE!")
    message(STATUS "${lto_error}")
  endif()
else()
  message(STATUS "")
  message(STATUS "Interprocedural deactivated for development purposes!")
  message(STATUS "To reactivate it, try disabling the option DEVEL_RELEASE.")
  message(STATUS "")
endif()
# add_compile_options(-fopenmp)



# # for sanitizing in debug mode:
# set(MY_SANITIZE_OPTIONS -fsanitize=address -fno-omit-frame-pointer -fsanitize=undefined)
# add_compile_options("$<$<CONFIG:DEBUG>:${MY_SANITIZE_OPTIONS}>")
# add_link_options("$<$<CONFIG:DEBUG>:${MY_SANITIZE_OPTIONS}>")



set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules)
set(ENV{CASADI_PREFIX} "${CMAKE_CURRENT_SOURCE_DIR}/casadiLocation")
find_package(CASADI REQUIRED)

add_subdirectory(src)

# # # IPOPT
# # # you can supply your own ipopt directory or build ipopt with cmake.
# # # The building in cmake up to now works only, if coinbrew works.
# if (NOT DEFINED IPOPT_DIRECTORY)

# ###############################################################
# # Ipopt building:
# ###############################################################

#   #fetch coinbrew for building ipopt:
  
#   if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/Ipopt/coinbrew")
#   execute_process(
#     COMMAND wget https://raw.githubusercontent.com/coin-or/coinbrew/master/coinbrew
#     WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/Ipopt"
#     )
#   execute_process(
#     COMMAND chmod u+x coinbrew
#     WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/Ipopt"
#     )
#   endif()
  
  
  
#   execute_process(
#     COMMAND ./coinbrew fetch Ipopt --no-prompt --skip=ThirdParty/ASL
#     WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/Ipopt"
#     )
  
  
#   ######################################
#   # Set up the include directories beforehand, because it seems difficult to supply
#   # include directories that are only generated in a build step.
#   ######################################
#   set(ipopt_library_headers ${CMAKE_CURRENT_BINARY_DIR}/Ipopt/dist/include/coin-or)
#   file(MAKE_DIRECTORY ${ipopt_library_headers})
    
#   # get the right library name, hopefully cross-platform:
#   set(ipopt_library_name ${CMAKE_SHARED_LIBRARY_PREFIX}ipopt${CMAKE_SHARED_LIBRARY_SUFFIX})
#   set(ipopt_library_file ${CMAKE_CURRENT_BINARY_DIR}/Ipopt/dist/lib/${ipopt_library_name})
  
  
  
#   # run coinbrew in parallel:
#   include(ProcessorCount)
#   ProcessorCount(NumberOfProcessors)

#   ########################################################################################################
#   # These hoops have to be jumped in order to stop cmake from inserting bad escape characters into environment variables.
#   ########################################################################################################
#   set(WITH_HIGHS ON)
#   set(WITH_BUILD_HIGHS ON)
#   set(WITH_IPOPT ON)
#   set(WITH_BUILD_IPOPT ON)
#   set(WITH_BUILD_MUMPS ON)
#   set(WITH_BUILD_METIS ON)


#   if(CMAKE_CXX_COMPILER_LAUNCHER MATCHES "ccache")
#     set(MY_OWN_CCACHE_VARIABLE "ccache")
#   endif()

#   # build casadi:
#   add_custom_command(OUTPUT  ${ipopt_library_file}
#     COMMAND ${CMAKE_COMMAND} -E env "ADD_FCFLAGS=${MY_FORTRAN_FLAGS}" "ADD_CFLAGS=${MY_C_FLAGS}" "ADD_CXXFLAGS=${MY_CXX_FLAGS}" "CC=${MY_OWN_CCACHE_VARIABLE} ${CMAKE_C_COMPILER}" "CXX=${MY_OWN_CCACHE_VARIABLE} ${CMAKE_CXX_COMPILER}" ./coinbrew build Ipopt --no-prompt --skip=ThirdParty/ASL --build-dir=${CMAKE_CURRENT_BINARY_DIR}/Ipopt --prefix=${CMAKE_CURRENT_BINARY_DIR}/Ipopt/dist --parallel-jobs ${NumberOfProcessors} || ${CMAKE_COMMAND} -E env "ADD_FCFLAGS=${MY_FORTRAN_FLAGS}" "ADD_CFLAGS=${MY_C_FLAGS}" "ADD_CXXFLAGS=${MY_CXX_FLAGS}" "CC=${MY_OWN_CCACHE_VARIABLE} ${CMAKE_C_COMPILER}" "CXX=${MY_OWN_CCACHE_VARIABLE} ${CMAKE_CXX_COMPILER}" ./coinbrew build Ipopt --no-prompt --skip=ThirdParty/ASL --build-dir=${CMAKE_CURRENT_BINARY_DIR}/Ipopt --prefix=${CMAKE_CURRENT_BINARY_DIR}/Ipopt/dist --parallel-jobs ${NumberOfProcessors}
#     WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Ipopt
#     USES_TERMINAL)
  
#   # create a target out of the ipopt compilation that depends on the generated library:
#   add_custom_target(ipopt_build_target ALL
#     DEPENDS ${ipopt_library_file}
#     )

# else()

#   find_library(ipopt_library_file
#     NAMES ipopt
#     PATHS "${IPOPT_DIRECTORY}/lib" NO_DEFAULT_PATH)
#   set(ipopt_library_headers "${IPOPT_DIRECTORY}/include/coin-or")
# endif()



#   # The ipopt target, on which we can depend in our own source files.
#   add_library(my_ipopt SHARED IMPORTED)
#   add_dependencies(my_ipopt ipopt_build_target ${ipopt_library_file})

#   set_target_properties(my_ipopt
#       PROPERTIES
#       IMPORTED_LOCATION ${ipopt_library_file}
#       )
#     target_include_directories(my_ipopt SYSTEM INTERFACE ${ipopt_library_headers})
  
#     # include_directories(SYSTEM ${ipopt_library_headers})

# 

add_subdirectory(docs)
add_subdirectory(tests)

