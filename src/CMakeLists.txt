cmake_minimum_required(VERSION 3.20)
project(CppProgram VERSION 0.1.1)
message(STATUS "OS: ${CMAKE_SYSTEM_NAME}")

# set the cpp and c standard for compilator:
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)


####################################################################################################
# BUILD A LIBRARY
include_directories(${CMAKE_SOURCE_DIR}/libs)
link_directories(${CMAKE_SOURCE_DIR}/libs)
add_library(test_lib STATIC
        ${CMAKE_SOURCE_DIR}/libs/hello.cpp
        ${CMAKE_SOURCE_DIR}/libs/hello.h
        )


find_package(Threads REQUIRED)


####################################################################################################
# COMPILE EXECUTABLE FILE
set(SRC_FILES main.cpp)
add_executable(${PROJECT_NAME} ${SRC_FILES})

target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_BINARY_DIR})


####################################################################################################
# SET COMPILER OPTIONS IF WINDOWS OR UNIX
if(WIN32)
        target_compile_options(${PROJECT_NAME} PRIVATE
                -fdiagnostics-color=always -Wall -Werror
                )
else()

        target_compile_options(${PROJECT_NAME} PRIVATE
                -stdlib=libc++ # default library for MacOS
                #-stdlib=libstdc++ # library usualy for linux and windows
                -fdiagnostics-color=always -Wall -Werror
                )
endif()


####################################################################################################
# SETTINGS FOR DEBUG BUILD
#set(CMAKE_BUILD_TYPE "Debug")
if (CMAKE_BUILD_TYPE MATCHES Debug)
if(WIN32)
                add_definitions(-D DEBUG=1
                        -D WINDOWS=1
                        -g
                        -O0
                        )
        else()
                #https://www.stackfinder.ru/questions/67512498/how-to-set-asan-options-environment-variable-in-cmake
                set(CMAKE_CXX_COMPILER_LAUNCHER ${CMAKE_COMMAND} 
                        -E env ASAN_OPTIONS=detect_stack_use_after_return=1 
                        -E env ASAN_OPTIONS=detect_leaks=1
                        ${CMAKE_CXX_COMPILER_LAUNCHER})

                add_definitions(-D DEBUG=1
                        -g
                        -fsanitize=address
                        -fsanitize=undefined
                        -O0
                        #-fno-omit-frame-pointer
                        #-fsanitize=leak
                        #-fno-optimize-sibling-calls
                        #-D_LIBCPP_DEBUG=1 # debuging macros turn on spec regime for libc++
                        #-D _GLIBCXX_DEBUG # debuging macros turn on spec regime for libstdc++
                        #-D _GLIBCXX_DEBUG_PEDANTIC # debuging macros turn on spec regime for libstdc++
                        )

                target_compile_options(${PROJECT_NAME}  PRIVATE 
                        -fsanitize=address
                        -fsanitize=undefined
                        #-fsanitize=leak
                        )
                        
                target_link_options(${PROJECT_NAME}  PRIVATE
                        -fsanitize=address
                        -fsanitize=undefined
                        #-fsanitize=leak
                        )
        endif()
endif()


####################################################################################################
# ADDINGS FOR MINGW
if(CMAKE_SYSTEM_NAME MATCHES "^MINGW")
    set(SYSTEM_LIBS -lstdc++)
else()
    set(SYSTEM_LIBS)
endif()


####################################################################################################
# LINKING LIBRARIES
target_link_libraries(${PROJECT_NAME} Threads::Threads ${SYSTEM_LIBS} test_lib)