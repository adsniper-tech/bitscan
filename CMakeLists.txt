cmake_minimum_required(VERSION 3.8.0)

project(bitscan VERSION 1.0.0 LANGUAGES C CXX)


if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type selected, default is Release")
    set(CMAKE_BUILD_TYPE "Release")
endif()

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

include(CTest)
enable_testing()

find_package(GTest REQUIRED)
include_directories(SYSTEM ${GTEST_INCLUDE_DIR})

find_package(GMock REQUIRED)
include_directories(SYSTEM ${GMOCK_INCLUDE_DIRS})



if(NOT DEFINED CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 11)
endif()

set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)

if(NOT DEFINED CMAKE_C_STANDARD)
    set(CMAKE_C_STANDARD 11)
endif()

set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)


enable_language(CXX)
include(CheckCXXCompilerFlag)

enable_language(C)
include(CheckCCompilerFlag)


check_cxx_compiler_flag("-std=gnu++11" COMPILER_KNOWS_CXX11)
if(COMPILER_KNOWS_CXX11)
    message ("[+] Use C++11")
    list(APPEND CMAKE_CXX_FLAGS "-std=gnu++11")
else()
    message("Compiler does not support -std=gnu++11")

    check_cxx_compiler_flag("-std=c++11" COMPILER_KNOWS_CXX11)
    if(COMPILER_KNOWS_CXX11)
        message ("[+] Use C++11")
        list(APPEND CMAKE_CXX_FLAGS "-std=c++11")
    else()
        message(FATAL_ERROR "Compiler does not support c++11")
    endif()
endif()


check_c_compiler_flag("-std=gnu11" COMPILER_KNOWS_C11)
if(COMPILER_KNOWS_C11)
    message ("[+] Use C11")
    list(APPEND CMAKE_C_FLAGS "-std=gnu11")
else()
    message("Compiler does not support -std=gnu11")

    check_c_compiler_flag("-std=c11" COMPILER_KNOWS_C11)
    if(COMPILER_KNOWS_C11)
        message ("[+] Use C11")
        list(APPEND CMAKE_C_FLAGS "-std=c11")
    else()
        message(FATAL_ERROR "Compiler does not support c11")
    endif()
endif()


message ("[+] Use custom C/C++ Flags")

set(COMMON_FLAGS "\
    -march=corei7-avx \
    -pipe \
    -m64 \
    -msse \
    -msse2 \
    -msse3 \
    -mssse3 \
    -msse4 \
    -msse4.1 \
    -msse4.2 \
    -mavx \
    -Wall \
    -Wextra \
    -Wformat-security \
    -fno-builtin-malloc \
    -fno-builtin-calloc \
    -fno-builtin-realloc \
    -fno-builtin-free \
    ")


set(COMMON_FLAGS_RELEASE "\
    -DNDEBUG \
    -s \
    -Ofast \
    -funroll-loops \
    -fomit-frame-pointer \
    -Wno-misleading-indentation \
    ")


if (CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    set(COMMON_FLAGS "\
        ${COMMON_FLAGS} \
         -pthread \
        ")
endif()


set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${COMMON_FLAGS}")
set(CMAKE_C_FLAGS_RELEASE "${COMMON_FLAGS_RELEASE}")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${COMMON_FLAGS}")
set(CMAKE_CXX_FLAGS_RELEASE "${COMMON_FLAGS_RELEASE}")



include_directories(.)


set(LIB_HEADERS
    bbalg.h  
    bbintrinsic.h
    bbintrinsic_sparse.h 
    bbobject.h  
    bbsentinel.h
    bbtypes.h  
    bitboard.h  
    bitboardn.h  
    bitboards.h  
    bitscan.h  
    config.h  
    tables.h
    )


set(LIB_SOURCES
    bbsentinel.cpp
    bitboard.cpp
    bitboardn.cpp
    bitboards.cpp
    tables.cpp
    )


set(LIB_TEST_SOURCES
    tests/test_bitblock.cpp  
    tests/test_bitstrings.cpp  
    tests/test_example.cpp  
    tests/test_masking.cpp  
    tests/test_sentinels.cpp  
    tests/test_sparse.cpp
   )


#Threads
find_package(Threads)


#boost
#set(Boost_USE_STATIC_LIBS ON)
#set(Boost_USE_MULTITHREADED ON)
#find_package(Boost COMPONENTS system filesystem REQUIRED)
#include_directories(${Boost_INCLUDE_DIRS})




set(EXT_LIBS
    ${CMAKE_THREAD_LIBS_INIT}
#    ${Boost_LIBRARIES}
    )


#APP Lib
add_library(${PROJECT_NAME} STATIC ${LIBS_HEADERS} ${LIB_SOURCES})
target_compile_options(${PROJECT_NAME} PRIVATE -Wno-unused-parameter)
target_link_libraries(${PROJECT_NAME} ${EXT_LIBS})


#Install
install(TARGETS ${PROJECT_NAME} ARCHIVE DESTINATION lib)
install(FILES ${LIB_HEADERS} DESTINATION include/bitscan/${LIB})


#Tests
add_executable (${PROJECT_NAME}_test ${LIB_TEST_SOURCES})
target_link_libraries(${PROJECT_NAME}_test ${PROJECT_NAME})
target_link_libraries(${PROJECT_NAME}_test ${GTEST_LIBRARIES} ${GMOCK_BOTH_LIBRARIES})
add_test(NAME ${PROJECT_NAME}_test COMMAND ${PROJECT_NAME}_test --gtest_output=xml:gtestresults.xml --gtest_color=yes)

