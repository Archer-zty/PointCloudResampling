cmake_minimum_required(VERSION 3.1)

project("PointCloudResampling")

if (NOT IS_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/ext/imgui/examples")
  message(FATAL_ERROR "3rd party is missing! "
    "You probably did not clone the project with --recursive. It is possible to recover "
    "by calling \"git submodule update --init --recursive\"")
endif()

if(NOT CMAKE_BUILD_TYPE)
	message(STATUS "No build type selected, Release as default")
	set(CMAKE_BUILD_TYPE "Release")
endif()

find_package(OpenMP)  
if(OPENMP_FOUND)  
	message("OpenMP FOUND")  
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
 elseif(OPENMP_FOUND) 
  message("OpenMP NOT found. Considering use a openmp-supported c compiler to enable openmp.")  
endif(OPENMP_FOUND)  

include(CheckCXXCompilerFlag)  
  CHECK_CXX_COMPILER_FLAG("-std=c++14" COMPILER_SUPPORTS_CXX14)  
  CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)  
  if(COMPILER_SUPPORTS_CXX14)  
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")  
  elseif(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")  
  else()  
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")  
endif() 

if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
endif()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

message(STATUS "operation system is ${CMAKE_SYSTEM}")
if (WIN32)
    message(STATUS "OS is windows")
    include_directories(ext/imgui)
    include_directories(ext/imgui/examples)
    include_directories(ext/imgui/examples/libs/gl3w)
    set(imgui_OS_src
        ext/imgui/examples/imgui_impl_glfw.cpp
        ext/imgui/examples/imgui_impl_opengl3.cpp
        ext/imgui/examples/libs/gl3w/GL/gl3w.c
    )
elseif (APPLE)
    message(STATUS "OS is Apple systens.")
    find_package(glfw3 REQUIRED)
    find_package(opengl REQUIRED)

    set(mac_extra_libs "")
    find_library(cocoa_library Cocoa)
    find_library(opengl_library OpenGL)
    find_library(corevideo_library CoreVideo)
    find_library(iokit_library IOKit)
    list(APPEND mac_extra_libs ${cocoa_library} ${opengl_library} ${corevideo_library} ${iokit_library})

    include_directories(ext/imgui)
    include_directories(ext/imgui/examples)
    include_directories(ext/imgui/examples/libs/gl3w)
    set(imgui_OS_src
        ext/imgui/examples/imgui_impl_glfw.cpp
        ext/imgui/examples/imgui_impl_opengl3.cpp
        ext/imgui/examples/libs/gl3w/GL/gl3w.c
    )
elseif (UNIX)
    message(STATUS "OS is UNIX-like OS's.")
    include_directories(ext/imgui)
    include_directories(ext/imgui/examples)
    include_directories(ext/imgui/examples/libs/gl3w)
    set(imgui_OS_src
        ext/imgui/examples/imgui_impl_glfw.cpp
        ext/imgui/examples/imgui_impl_opengl3.cpp
        ext/imgui/examples/libs/gl3w/GL/gl3w.c
    )
endif ()

#find_package(glfw3 REQUIRED)

set(imgui_src
    ext/imgui/imgui.cpp
    ext/imgui/imgui_demo.cpp
    ext/imgui/imgui_draw.cpp
    ext/imgui/imgui_widgets.cpp
)

set(src
  main.cpp
)

add_executable(PointCloudResampling ${imgui_OS_src} ${imgui_src} ${src})

if (WIN32)
    target_link_libraries(PointCloudResampling glfw3 opengl32 gdi32 shell32)
elseif (APPLE)
    target_link_libraries(PointCloudResampling glfw ${mac_extra_libs})
elseif (UNIX)
    target_link_libraries(PointCloudResampling glfw rt m dl GL)
endif ()

### necessary files
file(COPY res DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/ScreenShot)
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/RuntimeFiles)