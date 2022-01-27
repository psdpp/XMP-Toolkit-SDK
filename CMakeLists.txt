
# psdpp: top level CMakeLists to absorb all the custom steps

cmake_minimum_required(VERSION 3.22)

project(XMP-Toolkit-SDK)

include(ExternalProject)

ExternalProject_Add(XMPToolkitSDK
    SOURCE_DIR ${CMAKE_SOURCE_DIR}/build
    DOWNLOAD_COMMAND ""
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    INSTALL_COMMAND ""
)

if(${BUILD_SHARED_LIBS})
    set(XMP_BUILD_STATIC OFF)
    set(XMP_TARGET_NAME XMPCore)
    set(XMP_GEN_CHOICE 2)
else()
    set(XMP_BUILD_STATIC ON)
    set(XMP_TARGET_NAME XMPCoreStatic)
    set(XMP_GEN_CHOICE 3)
endif()

if (APPLE)
    ExternalProject_Add_Step(XMPToolkitSDK customConfigure
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        COMMAND echo ${XMP_GEN_CHOICE} | build/GenerateXMPToolkitSDK_mac.sh
        COMMENT "Generating Xcode project..."
        DEPENDEES patch
    )

    if (XMP_BUILD_STATIC)
        set(XMPXCODESCHEME XMPCoreStatic)
        set(XMPCONFIGURATION static)
    else(XMP_BUILD_STATIC)
        set(XMPXCODESCHEME XMPCore)
        set(XMPCONFIGURATION dynamic)
    endif(XMP_BUILD_STATIC)

    find_program(XMPXCPRETTY NAMES xcpretty)
    find_program(XMPXCBEAUTIFY NAMES xcbeautify)

    if(XMPXCPRETTY)
        set(XMPXCODEBUILDFILTER | xcpretty)
    elseif(XMPXCBEAUTIFY)
        set(XMPXCODEBUILDFILTER | xcbeautify)
    endif()

    set(XMPPROJECTFILE ${CMAKE_SOURCE_DIR}/build/xcode/${XMPCONFIGURATION}/intel_64_libcpp/XMPToolkitSDK64.xcodeproj)

    ExternalProject_Add_Step(XMPToolkitSDK customBuild
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        COMMAND xcodebuild
            -arch ${CMAKE_HOST_SYSTEM_PROCESSOR}
            -project ${XMPPROJECTFILE}
            -scheme ${XMPXCODESCHEME}
            -sdk ${CMAKE_OSX_SYSROOT}
            "GCC_PREPROCESSOR_DEFINITIONS=$GCC_PREPROCESSOR_DEFINITIONS ${XMPCOMPILEDEFINITIONS}"
            ${XMPXCODEBUILDFILTER}
        COMMENT "Building Xcode project..."
        DEPENDEES customConfigure
    )

elseif(UNIX)
    find_program(MAKE_EXE NAMES gmake nmake make)

    ExternalProject_Add_Step(XMPToolkitSDK customBuild
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/build
        COMMAND ${MAKE_EXE} all
        DEPENDEES customPatch
    )

elseif(WIN32)
    ExternalProject_Add_Step(XMPToolkitSDK customConfigure
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/build
        COMMAND echo ${XMP_GEN_CHOICE} | .\GenerateXMPToolkitSDK_win.bat
        DEPENDEES customPatch
    )

endif()
