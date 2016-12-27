cmake_minimum_required (VERSION 3.1)
project (cereal)

if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
  set(we_are_root ON)
else()
  set(we_are_root OFF)
endif()

find_package(Doxygen)

option(cereal_SKIP_PORTABILITY_TEST "Skip portability tests" OFF)
option(cereal_THREAD_SAFE "Use mutexes to ensure thread safety" OFF)
option(cereal_BUILD_UNIT_TESTS "Build the unit tests" ${we_are_root})
option(cereal_BUILD_SANDBOX "Build the sandbox executables" ${we_are_root})
option(cereal_BUILD_DOCUMENTATION "Build the documentation" ${DOXYGEN_FOUND})

add_library(cereal INTERFACE)
target_include_directories(cereal INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)
if(cereal_THREAD_SAFE)
  target_compile_definitions(cereal INTERFACE -DCEREAL_THREAD_SAFE=1)
  set(CEREAL_THREAD_LIBS "pthread")
else(cereal_THREAD_SAFE)
  set(CEREAL_THREAD_LIBS "")
endif(cereal_THREAD_SAFE)
if(NOT MSVC AND ${we_are_root})
  # Only turn on errors as warnings when we are the root project, otherwise we
  # pollute other projects that link with us.
  option(cereal_COMPILE_WITH_WERROR "Compile with -Werror option." ON)
  target_compile_options(cereal INTERFACE -Wall)
  target_compile_options(cereal INTERFACE -g)
  target_compile_options(cereal INTERFACE -Wextra)
  target_compile_options(cereal INTERFACE -Wshadow)
  target_compile_options(cereal INTERFACE -pedantic)
  if (cereal_COMPILE_WITH_WERROR)
    target_compile_options(cereal INTERFACE -Werror)
  endif(cereal_COMPILE_WITH_WERROR)
endif()

set(CMAKE_CXX_STANDARD_REQUIRED ON)
if (NOT DEFINED CMAKE_CXX_STANDARD OR CMAKE_CXX_STANDARD STREQUAL 98)
  set(CMAKE_CXX_STANDARD 11)
endif()

install(TARGETS cereal EXPORT cereal DESTINATION lib) # ignored
install(EXPORT cereal FILE cereal-config.cmake DESTINATION share/cmake/cereal)
install(DIRECTORY include/cereal DESTINATION include)

if(cereal_BUILD_UNIT_TESTS)
  if(we_are_root)
    enable_testing()
  endif(we_are_root)
  add_subdirectory(unittests)
endif(cereal_BUILD_UNIT_TESTS)

if(cereal_BUILD_SANDBOX)
  add_subdirectory(sandbox)
endif(cereal_BUILD_SANDBOX)

if(cereal_BUILD_DOCUMENTATION)
  configure_file("${CMAKE_CURRENT_SOURCE_DIR}/doc/doxygen.in" 
    "${CMAKE_CURRENT_BINARY_DIR}/doxygen.cfg" @ONLY)

  add_custom_target(doc
    COMMAND ${DOXYGEN_EXECUTABLE} 
      "${CMAKE_CURRENT_BINARY_DIR}/doxygen.cfg"
    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
    COMMENT "Generating API documentation with Doxygen" VERBATIM
    )

  configure_file("${CMAKE_CURRENT_SOURCE_DIR}/scripts/updatedoc.in" 
    "${CMAKE_CURRENT_BINARY_DIR}/updatedoc.sh" @ONLY)

  add_custom_target(update-doc
    COMMAND "${CMAKE_CURRENT_BINARY_DIR}/updatedoc.sh"
    DEPENDS doc
    COMMENT "Copying documentation to gh-pages branch" VERBATIM
    )
endif(cereal_BUILD_DOCUMENTATION)
