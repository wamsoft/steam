cmake_minimum_required(VERSION 3.16)

# UTF-8 source
add_compile_options("$<$<AND:$<C_COMPILER_ID:MSVC>,$<COMPILE_LANGUAGE:C>>:/utf-8>")
add_compile_options("$<$<AND:$<CXX_COMPILER_ID:MSVC>,$<COMPILE_LANGUAGE:CXX>>:/utf-8>")
add_compile_options("$<$<AND:$<CXX_COMPILER_ID:MSVC>,$<COMPILE_LANGUAGE:CXX>>:/Zc:__cplusplus>")

# MSVC static runtime
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

set(PROJECT_NAME krkrsteam)
set(PROJECT_VERSION 1.00)

project(${PROJECT_NAME} VERSION ${PROJECT_VERSION})

if(TVP_STATIC_PLUGIN)
    set(TVP_LIBRARY_TYPE STATIC)
else()
    set(TVP_LIBRARY_TYPE SHARED)
endif()

add_library(${PROJECT_NAME} ${TVP_LIBRARY_TYPE}
    ${CMAKE_CURRENT_SOURCE_DIR}/../ncbind/ncbind.cpp
    Main.cpp
    Storages.cpp
    SteamAchievements.cpp
)

if (TVP_STATIC_PLUGIN)
target_compile_definitions(${PROJECT_NAME} PRIVATE
    TVP_STATIC_PLUGIN
    TVP_PLUGIN_NAME=${PROJECT_NAME}
)
endif()

target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../ncbind
)

target_link_libraries(${PROJECT_NAME} PUBLIC
    tp_stub
    steam_api
)

# 環境変数 STEAMWORKS_SDK の設定を強制する
if(DEFINED ENV{STEAMWORKS_SDK})
    target_include_directories(${PROJECT_NAME} PUBLIC
        "$ENV{STEAMWORKS_SDK}/public/steam"
    )
    target_link_directories(${PROJECT_NAME} PUBLIC
        "$ENV{STEAMWORKS_SDK}/redistributable_bin"
    )
else()
    message(FATAL_ERROR "ERR: Please set environmental variable: STEAMWORKS_SDK")
endif()