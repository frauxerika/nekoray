cmake_minimum_required(VERSION 3.15)

project(nekobox VERSION 0.1 LANGUAGES CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# WINDOWS PDB FILE
if (WIN32)
    if (MSVC)
        set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /Zi")
        set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} /DEBUG /OPT:REF /OPT:ICF")
    endif ()
endif ()

# Find Qt
if (NOT QT_VERSION_MAJOR)
    set(QT_VERSION_MAJOR 5)
endif ()
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets Network Svg LinguistTools)

if (NKR_CROSS)
    set_property(TARGET Qt5::moc PROPERTY IMPORTED_LOCATION /usr/bin/moc)
    set_property(TARGET Qt5::uic PROPERTY IMPORTED_LOCATION /usr/bin/uic)
    set_property(TARGET Qt5::rcc PROPERTY IMPORTED_LOCATION /usr/bin/rcc)
    set_property(TARGET Qt5::lrelease PROPERTY IMPORTED_LOCATION /usr/bin/lrelease)
    set_property(TARGET Qt5::lupdate PROPERTY IMPORTED_LOCATION /usr/bin/lupdate)
endif ()

#### Platform Variables ####
if (WIN32)
    include("cmake/windows/windows.cmake")
else ()
    include("cmake/linux/linux.cmake")
endif ()

#### default prefix path ####

if (NOT NKR_LIBS)
    if (NKR_PACKAGE)
        list(APPEND NKR_LIBS ${CMAKE_SOURCE_DIR}/libs/deps/package)
    else ()
        list(APPEND NKR_LIBS ${CMAKE_SOURCE_DIR}/libs/deps/built)
    endif ()
endif ()

if (NOT NKR_DISABLE_LIBS)
    list(APPEND CMAKE_PREFIX_PATH ${NKR_LIBS})
endif ()

message("[CMAKE_PREFIX_PATH] ${CMAKE_PREFIX_PATH}")

# for some cross toolchain
list(APPEND CMAKE_FIND_ROOT_PATH ${CMAKE_PREFIX_PATH})
message("[CMAKE_FIND_ROOT_PATH] ${CMAKE_FIND_ROOT_PATH}")

#### NKR ####

include("cmake/print.cmake")
include("cmake/nkr.cmake")

find_package(Threads)

#### NKR EXTERNAL ####

if (NKR_NO_EXTERNAL)
    set(NKR_NO_GRPC 1)
    set(NKR_NO_YAML 1)
    set(NKR_NO_ZXING 1)
    set(NKR_NO_QHOTKEY 1)
endif ()

# grpc
if (NKR_NO_GRPC)
    nkr_add_compile_definitions(NKR_NO_GRPC)
else ()
    # My proto
    include("cmake/myproto.cmake")
    list(APPEND NKR_EXTERNAL_TARGETS myproto)
endif ()

# yaml-cpp
if (NKR_NO_YAML)
    nkr_add_compile_definitions(NKR_NO_YAML)
else ()
    find_package(yaml-cpp CONFIG REQUIRED) # only Release is built
    list(APPEND NKR_EXTERNAL_TARGETS yaml-cpp)
endif ()

# zxing-cpp
if (NKR_NO_ZXING)
    nkr_add_compile_definitions(NKR_NO_ZXING)
else ()
    find_package(ZXing CONFIG REQUIRED)
    list(APPEND NKR_EXTERNAL_TARGETS ZXing::ZXing)
endif ()

# QHotkey (static submodule)
if (NKR_NO_QHOTKEY)
    nkr_add_compile_definitions(NKR_NO_QHOTKEY)
else ()
    set(QHOTKEY_INSTALL OFF)
    set(BUILD_SHARED_LIBS OFF)
    set(CMAKE_POLICY_VERSION_MINIMUM 3.5)
    add_subdirectory(3rdparty/QHotkey)
    list(APPEND NKR_EXTERNAL_TARGETS qhotkey)
endif ()

#### debug print ####

if (DBG_CMAKE)
    print_all_variables()
    print_target_properties(myproto)
    print_target_properties(yaml-cpp)
    print_target_properties(ZXing::ZXing)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CMAKE_COMMAND} -E time")
    set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK "${CMAKE_COMMAND} -E time")
endif ()

# Sources
set(PROJECT_SOURCES
        ${PLATFORM_SOURCES}

        src/app/main.cpp
        src/app/NekoGui.cpp
        src/app/NekoGui_Utils.cpp
        src/app/HTTPRequestHelper.cpp

        3rdparty/base64.cpp
        3rdparty/qrcodegen.cpp
        3rdparty/QtExtKeySequenceEdit.cpp

        3rdparty/qv2ray/v2/ui/QvAutoCompleteTextEdit.cpp
        3rdparty/qv2ray/v2/components/proxy/QvProxyConfigurator.cpp
        3rdparty/qv2ray/v2/ui/widgets/common/QJsonModel.cpp
        3rdparty/qv2ray/v2/ui/widgets/editors/w_JsonEditor.cpp
        3rdparty/qv2ray/v2/ui/widgets/editors/w_JsonEditor.hpp
        3rdparty/qv2ray/v2/ui/widgets/editors/w_JsonEditor.ui

        3rdparty/qv2ray/v3/components/GeositeReader/GeositeReader.cpp
        3rdparty/qv2ray/v3/components/GeositeReader/picoproto.cpp

        src/services/rpc/gRPC.cpp

        src/core/database/Database.cpp
        src/core/database/traffic/TrafficLooper.cpp
        src/core/database/ProfileFilter.cpp
        src/core/database/ConfigBuilder.cpp

        src/core/config/AbstractBean.cpp
        src/core/config/Bean2CoreObj_box.cpp
        src/core/config/Bean2External.cpp
        src/core/config/Bean2Link.cpp
        src/core/config/Link2Bean.cpp
        src/core/config/ChainBean.hpp # translate

        src/services/subscription/GroupUpdater.cpp

        src/sys/ExternalProcess.cpp
        src/sys/AutoRun.cpp

        src/ui/ThemeManager.cpp
        src/ui/Icon.cpp

        src/ui/mainwindow_grpc.cpp
        src/ui/mainwindow.cpp
        src/ui/mainwindow.h
        src/ui/mainwindow.ui

        src/ui/edit/dialog_edit_profile.h
        src/ui/edit/dialog_edit_profile.cpp
        src/ui/edit/dialog_edit_profile.ui
        src/ui/edit/dialog_edit_group.h
        src/ui/edit/dialog_edit_group.cpp
        src/ui/edit/dialog_edit_group.ui

        src/ui/edit/edit_chain.h
        src/ui/edit/edit_chain.cpp
        src/ui/edit/edit_chain.ui
        src/ui/edit/edit_socks_http.h
        src/ui/edit/edit_socks_http.cpp
        src/ui/edit/edit_socks_http.ui
        src/ui/edit/edit_shadowsocks.h
        src/ui/edit/edit_shadowsocks.cpp
        src/ui/edit/edit_shadowsocks.ui
        src/ui/edit/edit_vmess.h
        src/ui/edit/edit_vmess.cpp
        src/ui/edit/edit_vmess.ui
        src/ui/edit/edit_trojan_vless.h
        src/ui/edit/edit_trojan_vless.cpp
        src/ui/edit/edit_trojan_vless.ui

        src/ui/edit/edit_naive.h
        src/ui/edit/edit_naive.cpp
        src/ui/edit/edit_naive.ui

        src/ui/edit/edit_quic.h
        src/ui/edit/edit_quic.cpp
        src/ui/edit/edit_quic.ui

        src/ui/edit/edit_custom.h
        src/ui/edit/edit_custom.cpp
        src/ui/edit/edit_custom.ui

        src/ui/dialog_basic_settings.cpp
        src/ui/dialog_basic_settings.h
        src/ui/dialog_basic_settings.ui

        src/ui/dialog_manage_groups.cpp
        src/ui/dialog_manage_groups.h
        src/ui/dialog_manage_groups.ui

        src/ui/dialog_manage_routes.cpp
        src/ui/dialog_manage_routes.h
        src/ui/dialog_manage_routes.ui

        src/ui/dialog_vpn_settings.cpp
        src/ui/dialog_vpn_settings.h
        src/ui/dialog_vpn_settings.ui

        src/ui/dialog_hotkey.cpp
        src/ui/dialog_hotkey.h
        src/ui/dialog_hotkey.ui

        src/ui/widget/ProxyItem.cpp
        src/ui/widget/ProxyItem.h
        src/ui/widget/ProxyItem.ui
        src/ui/widget/GroupItem.cpp
        src/ui/widget/GroupItem.h
        src/ui/widget/GroupItem.ui

        res/neko.qrc
        res/theme/feiyangqingyun/qss.qrc
        ${QV2RAY_RC}
)

# Qt exe
if (${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(nekobox
            MANUAL_FINALIZATION
            ${PROJECT_SOURCES}
    )
    # Define target properties for Android with Qt 6 as:
    #    set_property(TARGET nekobox APPEND PROPERTY QT_ANDROID_PACKAGE_SOURCE_DIR
    #                 ${CMAKE_CURRENT_SOURCE_DIR}/android)
    # For more information, see https://doc.qt.io/qt-6/qt-add-executable.html#target-creation
else ()
    if (ANDROID)
        add_library(nekobox SHARED
                ${PROJECT_SOURCES}
        )
        # Define properties for Android with Qt 5 after find_package() calls as:
        #    set(ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android")
    else ()
        add_executable(nekobox
                ${PROJECT_SOURCES}
        )
    endif ()
endif ()

# Target

set_property(TARGET nekobox PROPERTY AUTOUIC ON)
set_property(TARGET nekobox PROPERTY AUTOMOC ON)
set_property(TARGET nekobox PROPERTY AUTORCC ON)

set_target_properties(nekobox PROPERTIES
        WIN32_EXECUTABLE TRUE
)

target_include_directories(nekobox PRIVATE src)

# Target Source Translations

set(TS_FILES
        translations/zh_CN.ts
        translations/fa_IR.ts
        translations/ru_RU.ts
)
set(LUPDATE_OPTIONS
        -locations none -no-obsolete
)
if (${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_lupdate(nekobox TS_FILES ${TS_FILES} OPTIONS ${LUPDATE_OPTIONS})
    qt_add_lrelease(nekobox TS_FILES ${TS_FILES} QM_FILES_OUTPUT_VARIABLE QM_FILES)
else ()
    qt5_create_translation(QM_FILES ${PROJECT_SOURCES} ${TS_FILES} OPTIONS ${LUPDATE_OPTIONS})
endif ()
configure_file(translations/translations.qrc ${CMAKE_BINARY_DIR} COPYONLY)
target_sources(nekobox PRIVATE ${CMAKE_BINARY_DIR}/translations.qrc)

# Target Link

target_link_libraries(nekobox PRIVATE
        Qt${QT_VERSION_MAJOR}::Widgets Qt${QT_VERSION_MAJOR}::Network Qt${QT_VERSION_MAJOR}::Svg
        Threads::Threads
        ${NKR_EXTERNAL_TARGETS}
        ${PLATFORM_LIBRARIES}
)

if (QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(nekobox)
endif ()
