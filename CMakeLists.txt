cmake_minimum_required(VERSION 3.20)

project(CrusaderRPG VERSION 0.1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

include(FetchContent)

# Download and build SDL2 automatically so the user does not need to install
# SDL2 system-wide. Pin the version for reproducible builds.
set(SDL_TESTS OFF CACHE BOOL "" FORCE)
set(SDL_TEST_LIBRARY OFF CACHE BOOL "" FORCE)

FetchContent_Declare(
    SDL2
    URL https://github.com/libsdl-org/SDL/releases/download/release-2.32.10/SDL2-2.32.10.tar.gz
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
)
FetchContent_MakeAvailable(SDL2)

add_executable(CrusaderRPG
    src/main.cpp
)

# SDL2 uses SDL2main on Windows to provide the platform-specific application
# entry point and then call our normal C++ main function.
target_link_libraries(CrusaderRPG PRIVATE
    SDL2::SDL2main
    SDL2::SDL2
)

if (WIN32)
    target_link_libraries(CrusaderRPG PRIVATE opengl32)
elseif(APPLE)
    find_library(OPENGL_LIBRARY OpenGL)
    target_link_libraries(CrusaderRPG PRIVATE ${OPENGL_LIBRARY})
else()
    find_package(OpenGL REQUIRED)
    target_link_libraries(CrusaderRPG PRIVATE OpenGL::GL)
endif()

# When SDL2 is built as a shared library, copy its DLL beside our executable.
# This makes the Release build directly runnable from the build/Release folder.
if (WIN32)
    add_custom_command(TARGET CrusaderRPG POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            $<TARGET_FILE:SDL2::SDL2>
            $<TARGET_FILE_DIR:CrusaderRPG>
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            $<TARGET_FILE:SDL2::SDL2main>
            $<TARGET_FILE_DIR:CrusaderRPG>
        COMMENT "Copying SDL2 runtime files beside CrusaderRPG.exe"
    )
endif()

if (MSVC)
    target_compile_options(CrusaderRPG PRIVATE /W4)
else()
    target_compile_options(CrusaderRPG PRIVATE -Wall -Wextra -Wpedantic)
endif()
