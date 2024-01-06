cmake_minimum_required(VERSION 3.10.2)

set(addons ofxAndroid ofxAccelerometer ofxOpenCv)

project("ofLibProject")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-format-security")
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(openFrameworksProject_DIR "${ofRoot}/libs")
get_filename_component(openFrameworksProject_DIR "${openFrameworksProject_DIR}" ABSOLUTE)

find_library(openFrameworksProject openFrameworksProject)
include_directories(".")
include_directories(${openFrameworksProject_DIR})
file(GLOB_RECURSE HEADERS "${openFrameworksProject_DIR}/*.h")
file(GLOB_RECURSE SOURCES "${openFrameworksProject_DIR}/*.cpp")

macro(add_subdirectories_recursively root_dir)
    file(GLOB_RECURSE dirs LIST_DIRECTORIES true  "${root_dir}/*")
    foreach(dir ${dirs})
        if(IS_DIRECTORY "${dir}")
            include_directories("${dir}")
        endif()
    endforeach()
endmacro()
add_subdirectories_recursively("${openFrameworksProject_DIR}")

set(archive_files "")
file(GLOB_RECURSE archive_files "${openFrameworksProject_DIR}/*.a")

set(excludes
        "ofQtUtils.cpp" "ofQuickTimeGrabber.cpp" "ofQuickTimePlayer.cpp" "ofDirectShowGrabber.cpp"
        "ofDirectShowPlayer.cpp" "ofGstUtils.cpp" "ofGstVideoGrabber.cpp" "ofGstVideoPlayer.cpp"
        "ofAppGlutWindow.cpp" "ofAppEGLWindow.cpp" "ofAppGLFWWindow.cpp" "ofCairoRenderer.cpp"
        "ofFmodSoundPlayer.cpp" "ofOpenALSoundPlayer.cpp" "ofRtAudioSoundStream.cpp"
        "ofMediaFoundationSoundPlayer.cpp" "ofMediaFoundationPlayer.cpp" "ofGraphicsCairo.cpp"
        "glew" "videoInput" "fmod" "kiss" "assimp" "portaudio" "rtAudio" "poco/lib" "openssl/lib"
        "boost/include/boost" "openFrameworksCompiled" "dshow" "X11" "ofGst" "boost"
        "donotinclude" "libIlmImf.a"
        )

foreach(addon IN LISTS addons)
    file(GLOB_RECURSE ADDITIONAL_SOURCES "${openFrameworksProject_DIR}/../addons/${addon}/*.cpp")
    list(APPEND SOURCES ${ADDITIONAL_SOURCES})
    add_subdirectories_recursively("${openFrameworksProject_DIR}/../addons/${addon}/")
    file(GLOB_RECURSE additional_archive_files "${openFrameworksProject_DIR}/../addons/*.a")
    list(APPEND archive_files ${additional_archive_files})
endforeach ()

list(FILTER archive_files INCLUDE REGEX ".*${ANDROID_ABI}.*")
foreach(pattern ${excludes})
    list(FILTER SOURCES EXCLUDE REGEX ".*${pattern}.*")
    list(FILTER archive_files EXCLUDE REGEX ".*${pattern}.*")
endforeach()

add_library(${CMAKE_PROJECT_NAME} SHARED
        main.cpp
        ofApp.cpp
        ${SOURCES}
        )
set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES OUTPUT_NAME "OFAndroidApp")

set(ldf "-lm -lGLESv3 -lEGL -landroid  -lOpenSLES -lGLESv1_CM -ldl -llog -ffixed-x18")
target_link_libraries(${CMAKE_PROJECT_NAME} ${ldf})
target_link_libraries(${CMAKE_PROJECT_NAME} ${archive_files})
