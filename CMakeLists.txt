cmake_minimum_required(VERSION 3.13)
project(riff LANGUAGES C CXX)

option(RIFF_STATIC_LIBRARIES "If set to TRUE, will link libriff as a static library, dynamic otherwise. Default value is NOT(BUILD_SHARED_LIBS)." $<NOT:${BUILD_SHARED_LIBS}>)
option(RIFF_CXX_WRAPPER "If set to TRUE, will enable the C++ wrapper for libriff. Default is FALSE." FALSE)
option(RIFF_CXX_STD_FILESYSTEM_PATH "If set to TRUE, will enable support for std::filesystem::path arguments in the C++ wrapper for libriff. It is a C++17 feature and requires C++17 support in the host program, otherwise it only requires C++11. Does nothing without RIFF_CXX_WRAPPER set. Default is TRUE." TRUE)
option(RIFF_CXX_PRINT_ERRORS "If set to TRUE, will enable printing error messages to stdout from the C++ wrapper. Default is TRUE." TRUE)

if (RIFF_STATIC_LIBRARIES)
	add_library(riff STATIC "src/riff.c")
else()
	add_library(riff SHARED "src/riff.c")
endif()
target_include_directories(riff PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src/)
target_compile_features(riff PRIVATE c_std_99)
if (RIFF_CXX_WRAPPER)
	target_sources(riff PRIVATE "src/riff.cpp")
	target_compile_features(riff PUBLIC cxx_std_11)	# required for e.g. std::ios_base
	if (RIFF_CXX_STD_FILESYSTEM_PATH)
		target_compile_features(riff PUBLIC cxx_std_17)
		target_compile_definitions(riff PUBLIC RIFF_CXX17_SUPPORT=1)
	endif()
	if (RIFF_CXX_PRINT_ERRORS)
		target_compile_definitions(riff PRIVATE RIFF_CXX_PRINT_ERRORS=1)
	endif()
endif()

# examples and stuff
add_executable(example EXCLUDE_FROM_ALL examples/example.c)
target_link_libraries(example PRIVATE riff)
if (RIFF_CXX_WRAPPER)
	add_executable(cxx_example EXCLUDE_FROM_ALL examples/example.cpp)
	target_link_libraries(cxx_example PRIVATE riff)
endif()