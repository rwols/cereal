add_subdirectory(sandbox_shared_lib)

add_executable(sandbox sandbox.cpp)
target_link_libraries(sandbox PUBLIC cereal)
add_executable(sandbox_json sandbox_json.cpp)
target_link_libraries(sandbox_json PUBLIC cereal)
add_executable(sandbox_rtti sandbox_rtti.cpp)
target_link_libraries(sandbox_rtti PUBLIC cereal)

add_executable(sandbox_vs sandbox_vs.cpp)
target_link_libraries(sandbox_vs PUBLIC cereal sandbox_vs_dll)
target_include_directories(sandbox_vs PUBLIC sandbox_shared_lib)

find_package(Boost COMPONENTS serialization)
if(Boost_FOUND)
  add_executable(performance performance.cpp)
  target_include_directories(performance PUBLIC ${Boost_INCLUDE_DIRS})
  target_link_libraries(performance PUBLIC ${Boost_LIBRARIES} cereal)
endif(Boost_FOUND)
