cmake_minimum_required(VERSION 3.10)

project(leviosam CXX C)

set(PROJECT_URL "https://github.com/alshai/leviosam")
set(PROJECT_DESCRIPTION "leviosam: library for moving alignments between aligned genomes")
set(CMAKE_CXX_STANDARD 11)

set(HTS_LIB_DIR "" CACHE FILEPATH "path to htslib lib dir")
set(HTS_INC_DIR "" CACHE FILEPATH "path to htslib include dir")
set(SDSL_LIB_DIR "" CACHE FILEPATH "path to sdsl-lite lib dir")
set(SDSL_INC_DIR "" CACHE FILEPATH "path to sdsl-lite include dir")

if(EXISTS ${HTS_INC_DIR})
    include_directories(${HTS_INC_DIR})
endif()
if(EXISTS ${SDSL_INC_DIR})
    include_directories(${SDSL_INC_DIR})
endif()
if(EXISTS ${HTS_LIB_DIR})
    link_directories(${HTS_LIB_DIR})
endif()
if(EXISTS ${SDSL_LIB_DIR})
    link_directories(${SDSL_LIB_DIR})
endif()


find_library(HTS_LIB hts)
if(NOT HTS_LIB)
    message(FATAL_ERROR "htslib library not found. Please specify -D HTS_LIB_DIR and -D HTS_INC_DIR in the CMake command")
endif()

find_library(SDSL_LIB sdsl)
if(NOT SDSL_LIB)
    message(FATAL_ERROR "sdsl-lite library not found. Please specify -D SDSL_LIB_DIR and -D SDSL_INC_DIR in the CMake command")
endif()


# Download and unpack googletest at configure time
configure_file(CMakeLists.txt.in googletest-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
        RESULT_VARIABLE result
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/googletest-download )
if(result)
    message(FATAL_ERROR "CMake step for googletest failed: ${result}")
endif()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
        RESULT_VARIABLE result
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/googletest-download )
if(result)
    message(FATAL_ERROR "Build step for googletest failed: ${result}")
endif()

# Prevent overriding the parent project's compiler/linker
# settings on Windows
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

enable_testing()

# Add googletest directly to our build. This defines
# the gtest and gtest_main targets.
add_subdirectory(${CMAKE_CURRENT_BINARY_DIR}/googletest-src
                 ${CMAKE_CURRENT_BINARY_DIR}/googletest-build
                 EXCLUDE_FROM_ALL)

add_executable(leviosam leviosam.cpp bam_md.c bam_aux.c)
target_link_libraries(leviosam ${HTS_LIB})
target_link_libraries(leviosam ${SDSL_LIB})
target_link_libraries(leviosam pthread)
target_link_libraries(leviosam z)


add_executable(leviosam_test leviosam_test.cpp)
target_link_libraries(leviosam_test ${HTS_LIB})
target_link_libraries(leviosam_test ${SDSL_LIB})
target_link_libraries(leviosam_test gtest gtest_main)

add_test(NAME leviosam_test COMMAND leviosam_test
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/testdata
)

INSTALL(TARGETS leviosam DESTINATION bin)
INSTALL(FILES leviosam.hpp DESTINATION include)

