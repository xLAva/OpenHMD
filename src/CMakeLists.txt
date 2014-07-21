cmake_minimum_required(VERSION 2.6)

project(OpenHmd)

if(CMAKE_BUILD_TYPE STREQUAL "")
    set(CMAKE_BUILD_TYPE "RELEASE")
endif()



set(LIB_SOURCE_FILES
	fusion.c
	fusion.h
	log.h
	omath.h
	omath.c
	openhmd.c
	openhmdi.h
	platform.h
	platform-posix.c
	platform-win32.c
	drv_dummy/dummy.c
	drv_oculus_rift/packet.c
	drv_oculus_rift/rift.c
	drv_oculus_rift/rift.h
)

set(CUSTOM_COMPILE_FLAGS_ALL "")
set(LIBRARY_TYPE "SHARED")


if(WIN32)
    set(LIBRARY_NAME "openhmd")
else()
    set(LIBRARY_NAME "libopenhmd")
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/../include)

if(MSVC)
	set(HIDAPI_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../hidapi-0.7.0")
	include_directories(${HIDAPI_INCLUDE_DIR}/hidapi)

	add_library(imp_hidapi STATIC IMPORTED)
	set_property(TARGET imp_hidapi PROPERTY IMPORTED_LOCATION "${HIDAPI_INCLUDE_DIR}/windows/Release/hidapi.lib")
	set(EXTRA_LIBS ${EXTRA_LIBS} imp_hidapi)

	set(CUSTOM_COMPILE_FLAGS_ALL "${CUSTOM_COMPILE_FLAGS_ALL} -DDLL_EXPORT")
else()
    include(${CMAKE_CURRENT_SOURCE_DIR}/../FindHIDAPI.cmake)
	include_directories(${HIDAPI_INCLUDE_DIRS}/hidapi)


	set(CUSTOM_COMPILE_FLAGS_ALL "${CUSTOM_COMPILE_FLAGS_ALL} -std=c99")

    if(FORCE_I386)
        set(CUSTOM_COMPILE_FLAGS_ALL "${CUSTOM_COMPILE_FLAGS_ALL} -m32")
    	set(CMAKE_SHARED_LIBRARY_LINK_CXX_FLAGS "${CMAKE_SHARED_LIBRARY_LINK_CXX_FLAGS} -m32")
    endif()

    if(FORCE_STATIC)
        set(LIBRARY_TYPE "STATIC")
    endif()

	set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -g")
	set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${CUSTOM_COMPILE_FLAGS_ALL} -g")
endif()

set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} ${CUSTOM_COMPILE_FLAGS_ALL}")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} ${CUSTOM_COMPILE_FLAGS_ALL}")

set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} ${CUSTOM_COMPILE_FLAGS_ALL}")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${CUSTOM_COMPILE_FLAGS_ALL}")

add_library(${LIBRARY_NAME} ${LIBRARY_TYPE} ${LIB_SOURCE_FILES})
set_target_properties(${LIBRARY_NAME} PROPERTIES PREFIX "")

target_link_libraries(${LIBRARY_NAME} ${EXTRA_LIBS})
