cmake_minimum_required(VERSION 3.10)

set(LIBRARY_NAME iris_rendering_android)
set(LIBRARY_VERSION "0.1.0")

project(${LIBRARY_NAME} VERSION ${LIBRARY_VERSION} LANGUAGES CXX)

# Set C++ standard and enable exceptions/RTTI
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fexceptions -frtti")

set(SOURCES 
    "${CMAKE_CURRENT_SOURCE_DIR}/vm_util.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/iris_rtc_rendering_android.cc"
)

add_library(${LIBRARY_NAME} SHARED ${SOURCES})

find_library(log-lib log)
find_library(android-lib android)

# Updated target_link_libraries with all required libraries
target_link_libraries(${LIBRARY_NAME} PRIVATE
    ${log-lib}
    ${android-lib}
    GLESv2
    EGL
    atomic
    # Add these to resolve C++ standard library issues
    c++_shared
    dl
    # For NDK < r23, you might need this instead:
    # -static-libstdc++
)

# Support Android 15 16k page size
if (${ANDROID_ABI} STREQUAL "arm64-v8a" OR ${ANDROID_ABI} STREQUAL "x86_64")
  target_link_options(${LIBRARY_NAME} PRIVATE "-Wl,-z,max-page-size=16384")
  
  if(ANDROID_NDK_MAJOR LESS 22 AND NOT CMAKE_SHARED_LINKER_FLAGS MATCHES "-fuse-ld=lld")
      set_target_properties(${LIBRARY_NAME} PROPERTIES
          LINK_FLAGS "-Wl,-z,common-page-size=16384"
      )
  endif()
endif()

set(THIRD_PARTY_INCLUDE_DIRS
  "${CMAKE_CURRENT_SOURCE_DIR}/third_party/include/iris"
  "${CMAKE_CURRENT_SOURCE_DIR}/third_party/include/agora_rtc"
)

target_include_directories(${LIBRARY_NAME} PRIVATE
  "$<BUILD_INTERFACE:${THIRD_PARTY_INCLUDE_DIRS}>"
)
