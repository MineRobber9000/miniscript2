cmake_minimum_required(VERSION 3.12)
project(miniscript2 VERSION 2.0.0 LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(MS2_CORE_DIR ${CMAKE_SOURCE_DIR}/cpp/core)
set(MS2_GEN_DIR ${CMAKE_SOURCE_DIR}/generated)

file(GLOB MS2_CORE_SOURCES CONFIGURE_DEPENDS
    ${MS2_CORE_DIR}/*.cpp
    ${MS2_CORE_DIR}/*.c
)
list(FILTER MS2_CORE_SOURCES EXCLUDE REGEX "core/test_.*")
list(FILTER MS2_CORE_SOURCES EXCLUDE REGEX "core/debug_.*")

file(GLOB MS2_GEN_SOURCES CONFIGURE_DEPENDS
    ${MS2_GEN_DIR}/*.g.cpp
)

set(MINISCRIPT_SOURCES ${MS2_CORE_SOURCES} ${MS2_GEN_SOURCES})

set(EDITLINE_DIR ${CMAKE_SOURCE_DIR}/cpp/editline)

file(GLOB EDITLINE_SOURCES CONFIGURE_DEPENDS
	${EDITLINE_DIR}/*.c
)

add_executable(miniscript2
	${MINISCRIPT_SOURCES}
)

if(COSMOPOLITAN)
	# enable editline
	target_sources(miniscript2 PRIVATE ${EDITLINE_SOURCES})
	target_include_directories(miniscript2 PRIVATE ${EDITLINE_DIR})
	target_compile_definitions(miniscript2 PRIVATE USE_EDITLINE=1)
	# enable additional cosmopolitan features we use
	target_compile_options(miniscript2 PRIVATE
		-mcosmo
		-mtiny
		-Oz
	)
	# package libraries in ZipFS
	# InfoZIP needs us to have a file extension (or defaults to adding `.zip`)
	# cosmopolitan zen = use `.com` extension
	set_target_properties(miniscript2 PROPERTIES SUFFIX ".com")
	add_custom_command(TARGET miniscript2 POST_BUILD
		COMMAND zip -r -9 $<TARGET_FILE:miniscript2> lib -x "lib/sync_from_sysdisk.sh"
		WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
	)
endif()

target_include_directories(miniscript2 PRIVATE
    ${MS2_CORE_DIR}
    ${CMAKE_SOURCE_DIR}/cpp
    ${MS2_GEN_DIR}
)
