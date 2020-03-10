cmake_minimum_required(VERSION 2.8)
project(webdriverxx)

if (NOT MSVC)
	include(CheckCXXCompilerFlag)
	CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
	CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
	if(COMPILER_SUPPORTS_CXX11)
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
	elseif(COMPILER_SUPPORTS_CXX0X)
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
	endif()
endif()

enable_testing()

include_directories(include)
add_subdirectory(test)

install(DIRECTORY
		${CMAKE_SOURCE_DIR}/include/
		DESTINATION include/${PROJECT_NAME}
		FILES_MATCHING PATTERN "*")

