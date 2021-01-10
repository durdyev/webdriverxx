set(HEADER_FILES
	../include/webdriverxx.h 
	../include/webdriverxx/by.h 
	../include/webdriverxx/capabilities.h 
	../include/webdriverxx/client.h 
	../include/webdriverxx/client.inl 
	../include/webdriverxx/conversions.h 
	../include/webdriverxx/element.h 
	../include/webdriverxx/element.inl 
	../include/webdriverxx/errors.h 
	../include/webdriverxx/js_args.h 
	../include/webdriverxx/keys.h 
	../include/webdriverxx/response_status_code.h 
	../include/webdriverxx/session.h 
	../include/webdriverxx/session.inl 
	../include/webdriverxx/types.h 
	../include/webdriverxx/wait.h 
	../include/webdriverxx/wait_match.h 
	../include/webdriverxx/webdriver.h 
	../include/webdriverxx/window.h 
	../include/webdriverxx/picojson.h
	../include/webdriverxx/browsers/chrome.h
	../include/webdriverxx/browsers/firefox.h 
	../include/webdriverxx/browsers/ie.h 
	../include/webdriverxx/browsers/phantom.h
	../include/webdriverxx/detail/error_handling.h
	../include/webdriverxx/detail/factories.h 
	../include/webdriverxx/detail/factories_impl.h 
	../include/webdriverxx/detail/finder.h 
	../include/webdriverxx/detail/finder.inl 
	../include/webdriverxx/detail/http_client.h 
	../include/webdriverxx/detail/http_connection.h 
	../include/webdriverxx/detail/http_request.h 
	../include/webdriverxx/detail/keyboard.h 
	../include/webdriverxx/detail/meta_tools.h 
	../include/webdriverxx/detail/resource.h 
	../include/webdriverxx/detail/shared.h 
	../include/webdriverxx/detail/time.h 
	../include/webdriverxx/detail/to_string.h 
	../include/webdriverxx/detail/types.h 
	)

set(SOURCE_FILES
	alerts_test.cpp
	browsers_test.cpp
	capabilities_test.cpp
	conversions_test.cpp
	client_test.cpp
	element_test.cpp
	environment.h
	examples_test.cpp
	finder_test.cpp
	frames_test.cpp
	http_connection_test.cpp
	js_test.cpp
	keyboard_test.cpp
	main.cpp
	mouse_test.cpp
	resource_test.cpp
	session_test.cpp
	shared_test.cpp
	to_string_test.cpp
	wait_match_test.cpp
	wait_test.cpp
	webdriver_test.cpp
	)

file(COPY pages DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

add_definitions(-DWEBDRIVERXX_ENABLE_GMOCK_MATCHERS)

if (MSVC)
	add_definitions(-D_CRT_SECURE_NO_WARNINGS -D_SCL_SECURE_NO_WARNINGS)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4 /WX")
elseif (
	"${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR
	"${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU"
	)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror")
	if (NOT ${APPLE})
		set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-fsanitize=address -g -O1 -fno-omit-frame-pointer")
	endif()
endif()

# Coverage
if (UNIX)
	set(CMAKE_CONFIGURATION_TYPES ${CMAKE_CONFIGURATION_TYPES} Coverage)
	set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
		"Debug" "Release" "MinSizeRel" "RelWithDebInfo" "Coverage")
	set(CMAKE_CXX_FLAGS_COVERAGE "${CMAKE_CXX_FLAGS_DEBUG} --coverage")
	set(CMAKE_C_FLAGS_COVERAGE "${CMAKE_C_FLAGS_DEBUG} --coverage")

	find_program(LCOV_EXECUTABLE lcov)
	find_program(GENHTML_EXECUTABLE genhtml)

	add_custom_target(coverage
		${LCOV_EXECUTABLE} --directory=. --zerocounters
		COMMAND ${CMAKE_CTEST_COMMAND} -V
		COMMAND ${LCOV_EXECUTABLE} --capture --directory=. --output-file coverage.info
		COMMAND ${GENHTML_EXECUTABLE} coverage.info --output-directory coverage
		DEPENDS ${PROJECT_NAME}
		)
endif()

include(ExternalProject)

# CURL
if (WIN32)
	externalproject_add(curl_project
		URL http://curl.haxx.se/download/curl-7.55.0.tar.gz
		PREFIX curl
		CMAKE_ARGS -DCURL_STATICLIB=ON -DBUILD_SHARED_LIBS=OFF -DBUILD_CURL_EXE=OFF -DBUILD_CURL_TESTS=OFF -DCMAKE_USE_OPENSSL=OFF -DCURL_ZLIB=OFF -DHTTP_ONLY=ON
		INSTALL_COMMAND ""
		UPDATE_COMMAND ""
	)
	externalproject_get_property(curl_project source_dir binary_dir)
	include_directories("${source_dir}/include")
	link_directories("${binary_dir}/lib")
	list(APPEND LIBS libcurl wsock32 ws2_32)
	list(APPEND DEPS curl_project)
	add_definitions(-DCURL_STATICLIB)
else()
	find_package(CURL REQUIRED)
	include_directories(${CURL_INCLUDE_DIR})
	list(APPEND LIBS ${CURL_LIBRARIES})
endif()

# Google test
externalproject_add(googletest
		PREFIX googletest
		GIT_REPOSITORY https://github.com/google/googletest
		UPDATE_COMMAND ""
		INSTALL_COMMAND ""
)
externalproject_get_property(googletest source_dir binary_dir)
include_directories("${source_dir}/googletest/include")
include_directories("${source_dir}/googlemock/include")
link_directories("${binary_dir}/lib")
list(APPEND DEPS googletest)
list(APPEND LIBS gtest gmock)

# pthread
if (UNIX)
	set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
	find_package(Threads REQUIRED)
	list(APPEND LIBS ${CMAKE_THREAD_LIBS_INIT})
endif()

add_library(${PROJECT_NAME} ${SOURCE_FILES} ${HEADER_FILES})
add_dependencies(${PROJECT_NAME} ${DEPS})
target_link_libraries(${PROJECT_NAME} ${LIBS})

add_executable(${PROJECT_NAME}_test ${SOURCE_FILES} ${HEADER_FILES})
add_dependencies(${PROJECT_NAME}_test ${DEPS})
target_link_libraries(${PROJECT_NAME}_test ${LIBS})
add_test(${PROJECT_NAME}_test ${PROJECT_NAME})

if (APPLE)
	set_target_properties(${PROJECT_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME} SUFFIX ".dylib")
endif()

if (UNIX AND NOT APPLE)
	set_target_properties(${PROJECT_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME} SUFFIX ".so")
endif()

if (WIN32)
	set_target_properties(${PROJECT_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME} SUFFIX ".dll")
endif()

install(TARGETS ${PROJECT_NAME} DESTINATION lib)
