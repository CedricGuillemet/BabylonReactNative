cmake_minimum_required(VERSION 3.13.2)
# set targetPlatform, will be passed in from gradle when this sample is completed
# openGL Supportability
# platform         status
#   (0 12)           ES2/ES3 not supported
#   [12, 18)         ES2 only; for ES3, app do dynamic load/detection
#                    this applies to the situations that:
#                        - minimum API is set to less than 18. In this case
#                          there is no ES3 header/lib support inside NDK
#                        - the built APK might be running on newer API phones
#                    with dynamic loading of ES3, the same APK would still be able
#                    to use ES3. Otherwise, app would stuck with ES2 even phone is
#                    is newer than the minimum API level (for example, Android-27 etc).
#
#   [18, 24)         ES2 & ES3
#                    If app is built to only support API-18 or later,
#                    set minimum api level to 18 is good enough, NDK supprts ES3
#                    with the right header and lib files. No need to use ES3 dynamic
#                    detection.
#   [24, infinite)   ES2 & ES3 & Vulkan
project(ReactNativeBabylon)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall")
set(BABYLON_NATIVE_PLATFORM "Android")
set(OPENGL_LIB GLESv3)
set(CMAKE_CXX_EXTENSIONS OFF)
set(ANDROID_STL "c++_shared")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")

#message(SEND_ERROR "ANDROID_JSC_LIBPATH ${ANDROID_JSC_LIBPATH}/${ANDROID_ABI}/libjsc.so")
#message(SEND_ERROR "ANDROID_JSC_INCPATH ${ANDROID_JSC_INCPATH}")

# configure Babylon Native to use JSC (specifically the one that is part of the React Native application)
set(NAPI_JAVASCRIPT_ENGINE "JavaScriptCore" CACHE STRING "The JavaScript engine to power N-API")
add_library(javascript_engine SHARED IMPORTED GLOBAL)
set_target_properties(javascript_engine PROPERTIES IMPORTED_LOCATION "${ANDROID_JSC_LIBPATH}/${ANDROID_ABI}/libjsc.so")
target_include_directories(javascript_engine INTERFACE "${ANDROID_JSC_INCPATH}")

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/src/")

set(BABYLON_NATIVE_DIR "${CMAKE_CURRENT_LIST_DIR}/../submodules/BabylonNative")
add_subdirectory(${BABYLON_NATIVE_DIR}/Dependencies/ ${BABYLON_NATIVE_DIR}/build/Android_${CMAKE_ANDROID_ARCH_ABI}/Dependencies/)
add_subdirectory(${BABYLON_NATIVE_DIR}/Core/ ${BABYLON_NATIVE_DIR}/build/Android_${CMAKE_ANDROID_ARCH_ABI}/Core/)
add_subdirectory(${BABYLON_NATIVE_DIR}/Plugins/ ${BABYLON_NATIVE_DIR}/build/Android_${CMAKE_ANDROID_ARCH_ABI}/Plugins/)
# add_subdirectory(${BABYLON_NATIVE_DIR}) // TODO: We can probably switch to just adding the Babylon Native top level dir instead of the individual folders.

add_library(BabylonNativeJNI SHARED
            src/main/cpp/BabylonNativeJNI.cpp)

target_compile_definitions(BabylonNativeJNI PRIVATE NODE_ADDON_API_DISABLE_DEPRECATED)

# add_definitions(-DANDROID_STL=c++_shared)

target_link_libraries(BabylonNativeJNI
        ${OPENGL_LIB}
        android
        EGL
        log
        -lz
        JsRuntime
        NativeWindow
        NativeEngine)

# configure_file(
#     "${ANDROID_NDK}/sources/cxx-stl/llvm-libc++/libs/${ANDROID_ABI}/libc++_shared.so"
#     "${CMAKE_CURRENT_LIST_DIR}/src/main/jniLibs/${CMAKE_ANDROID_ARCH_ABI}/libc++_shared.so"
#     COPYONLY)